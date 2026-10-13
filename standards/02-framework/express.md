# Framework Standard: Express (Tier 1)

## 1. Purpose
Defines the declarative routing, functional controller dispatch, middleware chaining, and strict enum-first architecture for Express applications.

---

## 2. Applies when
- `framework: express` is selected in Step 2.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Express Standard |
|:---|:---|
| Step 1 (Architecture) | In microservices, applies to each service in `services/*`. In monolith, applies to the root app. |
| Step 8 (Shared Code) | Imports `registerRoutes`, `handleRequestWithResponse`, enums, and error handlers from `shared-utils`. |

---

## 4. Contract

### Declarative Route Registration
- Every domain router MUST define routes as an array of `RouteDefinition` objects and register them via `registerRoutes(router, routes)`:
  - Route paths and methods MUST NOT be attached imperatively with raw inline calls like `router.get('/', handler)`.
  - HTTP methods MUST be referenced using `HTTPRequestMethodsEnumEntity` (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
  - Route paths MUST use enums or constants from `shared-utils`.

### Functional Controller Request Handlers
- Controller methods MUST NOT be procedural `(req, res, next)` blocks with manual try/catch boilerplate.
- Controller endpoints MUST be wrapped using `handleRequestWithResponse`:
  ```typescript
  // Wraps service execution and forwards caught errors to Express error pipeline
  public getUser = handleRequestWithResponse((req, res) => this.userService.getUser(req, res));
  ```

### Strict Enum-First Architecture
- Magic strings, static response messages, and hardcoded numbers are STRICTLY PROHIBITED.
- Code MUST use:
  - `HTTPRequestMethodsEnumEntity` for HTTP verbs.
  - `HttpStatusCodesEnumEntity` for response status codes.
  - `CommonMessagesEnumEntity` for API response messages.
  - `NodeEnvEnum` for environments (`DEVELOPMENT`, `PRODUCTION`, `TEST`).
  - `LogLevelEnum` for logging thresholds.

### Single Dynamic Health Endpoint
- Every service MUST expose exactly ONE health check endpoint: `GET /health`.
- The endpoint MUST perform an active database connectivity probe (e.g. Mongoose `readyState === 1` or SQL `SELECT 1`) and return dynamic status. Dual `/health/live` and `/health/ready` endpoints are prohibited.

### One-Line Function Documentation
- Every exported function, middleware, utility, and handler MUST have a concise 1-line comment explaining its purpose.

---

### Route-Level Validation Middleware
- Payloads MUST be validated at the route boundary using `validateData(schema)` placed in `route.middlewares`.
- Services and controllers MUST NOT invoke `.safeParse()` or `.parse()`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Declarative routing with validateData middleware and functional controller dispatch
const routes: RouteDefinition[] = [
  {
    path: UserRoutesEnum.CREATE_USER,
    method: HTTPRequestMethodsEnumEntity.POST,
    middlewares: [validateData(createUserSchema)],
    handler: userController.createUser
  }
];
registerRoutes(router, routes);
export default router;
```

---

## 6. Agent Output Checklist
- [ ] Routes declared via `RouteDefinition[]` and mounted with `registerRoutes`.
- [ ] Input payloads validated via `validateData(schema)` in route middlewares array.
- [ ] Controllers wrap service executions with `handleRequestWithResponse`.
- [ ] Single `GET /health` endpoint verifying database connection.
- [ ] All HTTP methods, status codes, and messages use enums from `shared-utils`.
- [ ] One-line comments on all reusable functions and middlewares.

---

## 7. Common Mistakes
- **Imperative Route Binding:** Calling `router.post('/path', ...)` directly instead of using `registerRoutes`.
- **Manual Try/Catch in Controllers:** Writing duplicate `try { ... } catch(e) { next(e) }` in every controller action.
- **Using String Literals for HTTP Verbs:** Writing `'post'` or `'POST'` instead of `HTTPRequestMethodsEnumEntity.POST`.
- **Dual Health Endpoints:** Creating separate static liveness and readiness endpoints.
