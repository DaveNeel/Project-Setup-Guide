# Inter-Service Communication Standard: Synchronous REST (Tier 1)

## 1. Purpose
Defines contracts for synchronous HTTP/REST communication between microservices, covering client wrappers, correlation tracking, retries, and timeouts.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1, AND
- `inter_service_comm: rest` is selected in Step 7.

---

## 3. Interaction Notes

| Preceding Decision | Impact on REST Communication |
|:---|:---|
| Step 2 (Framework) | Uses NestJS `HttpModule` (Axios) or Express HTTP client wrapper. |
| Cross-Cutting | Propagates `x-correlation-id` and consumes the universal error envelope. |

---

## 4. Contract

### Client Wrapper & Configuration
- Outbound HTTP requests to sibling microservices MUST be executed via a dedicated, typed HTTP client wrapper.
- Sibling service base URLs MUST be resolved from typed environment variables (e.g. `AUTH_SERVICE_URL`, `CATALOG_SERVICE_URL`).
- Hardcoding hostnames, IP addresses, or ports in application code is STRICTLY PROHIBITED.

### Timeouts & Retries
- Outbound HTTP calls MUST enforce an explicit timeout threshold (default: 5,000ms).
- Idempotent HTTP methods (`GET`, `HEAD`) MAY implement automatic retries (maximum 3 attempts) with exponential backoff and jitter.
- Non-idempotent HTTP methods (`POST`, `PATCH`) MUST NOT retry automatically unless an `Idempotency-Key` header is explicitly attached.

### Header Propagation & Error Handling
- The incoming `x-correlation-id` header MUST be forwarded in every outbound HTTP request header.
- Downstream errors returning the standard error envelope MUST be captured and remapped to domain exceptions without losing the original error code.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Inter-service HTTP client header propagation
@Injectable()
export class ServiceHttpClient {
  constructor(private readonly http: HttpService) {}

  async call(url: string, correlationId: string): Promise<ApiResponse<T>> {
    const res = await firstValueFrom(
      this.http.get(url, {
        headers: { 'x-correlation-id': correlationId },
        timeout: 5000
      })
    );
    return res.data;
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Dedicated HTTP service client wrapper with configurable timeout.
- [ ] Outbound headers attach `x-correlation-id`.
- [ ] Environment variables define URLs for all target sibling services.

---

## 7. Common Mistakes
- **Unbounded HTTP Timeouts:** Making calls with no timeout, causing worker threads to hang indefinitely when downstream services stall.
- **Dropping Correlation IDs:** Generating a new correlation ID for outbound calls instead of forwarding the existing trace.
- **Unsafe Retries on POST:** Automatically retrying mutating POST requests, resulting in duplicate transactions.
