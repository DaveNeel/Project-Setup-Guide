# Cross-Cutting Standard: Structured Logging & Correlation IDs

## 1. Purpose
Defines contracts for structured JSON logging, correlation ID propagation, and sensitive data redaction across all services and frameworks.

---

## 2. Applies when
- All projects across all frameworks and topologies.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Logging Implementation |
|:---|:---|
| Step 2 (Framework) | NestJS: custom `LoggerService` wrapping Pino.<br>Express: `pino-http` middleware. |
| Step 9 (Config) | Configures log level via `LOG_LEVEL` environment variable. |

---

## 4. Contract

### Mandatory Log Record Schema
All logs emitted to standard output MUST be single-line JSON records containing these keys:
```json
{
  "timestamp": "2026-10-09T17:00:00.000Z",
  "level": "info",
  "service": "order-service",
  "correlationId": "00000000-0000-0000-0000-000000000000",
  "context": "OrderService",
  "message": "Order created successfully",
  "err": null
}
```

### Correlation ID Middleware
- An HTTP middleware MUST execute for every incoming request:
  - Check for inbound header `x-correlation-id`.
  - If present, validate and retain.
  - If absent or invalid, generate a new UUIDv4 via `crypto.randomUUID()`.
  - Set the `x-correlation-id` on the outgoing HTTP response header.
  - Bind `correlationId` to the active request context for logger injection.

### Sensitive Data Redaction
- Loggers MUST configure field redaction for keys: `password`, `token`, `secret`, `authorization`, `credit_card`, `apiKey`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Correlation ID middleware contract
export function correlationIdMiddleware(req: Request, res: Response, next: NextFunction) {
  const correlationId = (req.headers['x-correlation-id'] as string) || crypto.randomUUID();
  req.correlationId = correlationId;
  res.setHeader('x-correlation-id', correlationId);
  next();
}
```

---

## 6. Agent Output Checklist
- [ ] Pino logger configured with JSON output formatting.
- [ ] Correlation ID middleware mounted before route handlers.
- [ ] `x-correlation-id` header set on outgoing responses.
- [ ] Redaction configured for sensitive fields.

---

## 7. Common Mistakes
- **Using `console.log()`:** Emitting unstructured plain text strings instead of JSON.
- **Dropping Correlation ID in Async Workflows:** Failing to pass `correlationId` across background task invocations.
- **Logging Plaintext Passwords:** Printing raw request bodies containing `password` fields.
