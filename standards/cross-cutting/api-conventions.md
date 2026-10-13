# Cross-Cutting Standard: REST API Conventions & Response Envelopes

## 1. Purpose
Defines universal URI naming, routing conventions, the standard `APIResponseEntity<T>` envelope, and strict enum usage for HTTP methods and status codes.

---

## 2. Applies when
- All HTTP endpoints across all services.

---

## 3. Interaction Notes

| Preceding Decision | Impact on API Conventions |
|:---|:---|
| Step 2 (Framework) | Express routes registered via `registerRoutes(router, routes)`. |
| Step 8 (Shared Code) | Consumes `APIResponseEntity<T>`, `HTTPRequestMethodsEnumEntity`, and `HttpStatusCodesEnumEntity` from `shared-utils`. |

---

## 4. Contract

### URI Versioning & Routing Conventions
- All business endpoints MUST be mounted under `/api/v1/`.
- Routes MUST be declared via `RouteDefinition[]` utilizing `HTTPRequestMethodsEnumEntity` for verbs.
- Direct string literals for HTTP methods (e.g. `'get'`, `'post'`) are STRICTLY PROHIBITED.

### Universal API Response Entity (`APIResponseEntity<T>`)
Every HTTP response returned by any endpoint MUST conform strictly to this envelope:
```typescript
export interface APIResponseEntity<T = unknown> {
  status: boolean;
  statusCode: number;
  message: string;
  data: T;
}
```

#### Success Response Schema (2xx)
```json
{
  "status": true,
  "statusCode": 200,
  "message": "SUCCESS",
  "data": {}
}
```
- Status codes in controllers and formatters MUST use `HttpStatusCodesEnumEntity`.
- Response messages MUST use `CommonMessagesEnumEntity` rather than inline strings.

### One-Line Function Documentation
- Every response formatter, route definition array, and handler wrapper MUST include a 1-line explanatory comment.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Wraps arbitrary payload data and status code into standard APIResponseEntity envelope
export function createSuccessResponse<T>(
  data: T,
  statusCode: HttpStatusCodesEnumEntity = HttpStatusCodesEnumEntity.OK,
  message: CommonMessagesEnumEntity | string = CommonMessagesEnumEntity.SUCCESS
): APIResponseEntity<T> {
  return {
    status: true,
    statusCode,
    message,
    data
  };
}
```

---

## 6. Agent Output Checklist
- [ ] Responses typed with `APIResponseEntity<T>` (`status`, `statusCode`, `message`, `data`).
- [ ] HTTP verbs referenced exclusively via `HTTPRequestMethodsEnumEntity`.
- [ ] Status codes referenced exclusively via `HttpStatusCodesEnumEntity`.
- [ ] 1-line comments present on all handlers and formatters.

---

## 7. Common Mistakes
- **Using Legacy Envelopes:** Emitting `{ success: true, data }` instead of `{ status: true, statusCode, message, data }`.
- **Hardcoding HTTP Status Numbers:** Using raw numbers like `200` or `201` instead of enum properties.
- **Unformatted Responses:** Emitting raw un-enveloped JSON objects.
