# Cross-Cutting Standard: Centralized Error Handling

## 1. Purpose
Defines the universal error envelope matching `APIResponseEntity`, HTTP status mappings, and exception handling pipeline required across all frameworks.

---

## 2. Applies when
- All projects across all frameworks and topologies.

---

## 3. Interaction Notes

| Framework | Implementation Location |
|:---|:---|
| NestJS | `src/common/filters/all-exceptions.filter.ts` wired via `app.useGlobalFilters()`. |
| Express | `src/common/middleware/error-handler.middleware.ts` mounted as final 4-arg middleware. |

---

## 4. Contract

### The Universal Error Response Envelope
Every error response returned by any endpoint MUST conform exactly to the `APIResponseEntity` schema:
```json
{
  "status": false,
  "statusCode": 400,
  "message": "Human readable error description",
  "data": {}
}
```

### HTTP Status Code & Error Code Mappings
- **400 Bad Request:** `statusCode: 400`. Validation failure message from `validateData` or bad input.
- **401 Unauthorized:** `statusCode: 401`. `message: "UNAUTHORIZED"` or invalid credentials.
- **403 Forbidden:** `statusCode: 403`. `message: "FORBIDDEN"`.
- **404 Not Found:** `statusCode: 404`. `message: "RESOURCE_NOT_FOUND"`.
- **409 Conflict:** `statusCode: 409`. `message: "CONFLICT"`.
- **500 Internal Server Error:** `statusCode: 500`. `message: "INTERNAL_SERVER_ERROR"`.
  - In production (`NODE_ENV === 'production'`), `message` MUST be generic: `"An unexpected internal error occurred"`.
  - Stack traces MUST NEVER be leaked into the response envelope.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Central error middleware generating standard APIResponseEntity error envelopes
export function errorHandler(err: Error, req: Request, res: Response, _next: NextFunction) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      status: false,
      statusCode: err.statusCode,
      message: err.message,
      data: {}
    });
  }
  return res.status(HttpStatusCodesEnumEntity.INTERNAL_SERVER_ERROR).json({
    status: false,
    statusCode: HttpStatusCodesEnumEntity.INTERNAL_SERVER_ERROR,
    message: CommonMessagesEnumEntity.INTERNAL_SERVER_ERROR,
    data: {}
  });
}
```

---

## 6. Agent Output Checklist
- [ ] Error middleware returns `{ status: false, statusCode, message, data: {} }`.
- [ ] Correlation ID is extracted and set on response header `x-correlation-id`.
- [ ] Production 500 errors hide stack traces and internal exception messages.

---

## 7. Common Mistakes
- **Returning Legacy Error Schemas:** Emitting `{ success: false, error: { ... } }` instead of `APIResponseEntity`.
- **Leaking Stack Traces:** Including `err.stack` in API responses in production environments.
- **Missing Status Code in JSON:** Omitting `statusCode` field from error body.
