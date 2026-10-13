# Cross-Cutting Standard: Route-Level Zod Validation Middleware

## 1. Purpose
Defines rules for request payload validation via route-level middleware, isolating schemas in modules and preventing duplicate validation boilerplate inside service logic.

---

## 2. Applies when
- All HTTP endpoints accepting a request body, query parameters, or URL route params.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Validation |
|:---|:---|
| Step 2: `express` | Validates payloads via `validateData(schema)` middleware in route definitions. |
| Step 8: `shared-utils` | Exports `validateData` middleware and `APIResponseEntity<T>` types. |

---

## 4. Contract

### Schema Placement & Service Decoupling
- Zod schemas MUST be declared at the microservice module level inside `<module>.dto.ts`.
- Validation MUST be performed exclusively in route middleware using `validateData(schema)`.
- Service methods MUST NOT call `schema.safeParse()` or `schema.parse()`. Services receive pre-validated data directly on `req.body`.

### Validation Middleware Implementation
- `validateData(schema)` MUST parse `req.body` using `schema.parse(req.body)`:
  - On success: calls `next()`.
  - On `ZodError`: extracts `firstIssue?.message || 'Validation failed'` and returns HTTP 400.
  - On unknown error: returns HTTP 500 with `CommonMessagesEnumEntity.INTERNAL_SERVER_ERROR`.
- The error response MUST strictly conform to `APIResponseEntity`:
  ```json
  {
    "status": false,
    "statusCode": 400,
    "message": "Validation failed",
    "data": {}
  }
  ```

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Validates incoming request payload against a Zod schema before hitting the controller
export function validateData(schema: z.ZodTypeAny) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        const firstIssue = error.issues[0];
        const errorMessage = firstIssue?.message || 'Validation failed';
        return res.status(HttpStatusCodesEnumEntity.BAD_REQUEST).json({
          status: false,
          statusCode: HttpStatusCodesEnumEntity.BAD_REQUEST,
          message: errorMessage,
          data: {},
        });
      }
      return res.status(HttpStatusCodesEnumEntity.INTERNAL_SERVER_ERROR).json({
        status: false,
        statusCode: HttpStatusCodesEnumEntity.INTERNAL_SERVER_ERROR,
        message: CommonMessagesEnumEntity.INTERNAL_SERVER_ERROR,
        data: {},
      });
    }
  };
}
```

---

## 6. Agent Output Checklist
- [ ] Zod schemas defined in service module `<module>.dto.ts`.
- [ ] `validateData(schema)` attached to route `middlewares` array.
- [ ] Zero calls to `.safeParse()` or `.parse()` inside service files.
- [ ] Validation errors return `{ status: false, statusCode: 400, message, data: {} }`.

---

## 7. Common Mistakes
- **Validating Inside Services:** Calling `schema.safeParse(req.body)` inside domain service methods.
- **Leaking Raw Zod Error Trees:** Emitting unformatted Zod arrays instead of `APIResponseEntity`.
- **Missing Middleware in Route Definition:** Omitting `validateData` from the route's `middlewares` array.
