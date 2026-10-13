# Shared Code Standard: Shared Utils Package (Tier 1)

## 1. Purpose
Defines structure, packaging, and contents for `shared-utils`, establishing reusable enums, routing helpers, error pipelines, and standalone repository readiness.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1, AND
- `shared_code_strategy: shared-utils` is selected in Step 8.

---

## 3. Interaction Notes

| Preceding Decision | Impact on shared-utils |
|:---|:---|
| Step 1 (Architecture) | In microservices, located under `packages/shared-utils` (or consumed from a dedicated repository). |
| Step 2 (Framework) | Houses `registerRoutes`, `handleRequestWithResponse`, and Express middleware helpers. |

---

## 4. Contract

### Package Naming & Standalone Repository Readiness
- The shared library MUST be named `shared-utils`.
- It MUST be structured with an independent `package.json`, `tsconfig.json`, and `src/` tree so it can be published or extracted to an independent external git repository without code changes.

### Mandatory Contents of `shared-utils`
1. **Universal Enums (Strict No-String-Literal Rule):**
   - `HTTPRequestMethodsEnumEntity`: Typed HTTP verbs (`get`, `post`, `put`, `delete`, `patch`).
   - `HttpStatusCodesEnumEntity`: Standard HTTP status numbers (`200`, `201`, `400`, `401`, `404`, `500`).
   - `CommonMessagesEnumEntity`: Standard response messages (e.g. `RESOURCE_NOT_FOUND`, `INTERNAL_SERVER_ERROR`).
   - `NodeEnvEnum`: Environment modes (`development`, `production`, `test`).
   - `LogLevelEnum`: Log levels (`debug`, `info`, `warn`, `error`).
   - `HealthStatusEnum`: Health states (`ok`, `error`, `up`, `down`).
2. **Declarative Routing & Controller Dispatch:**
   - `registerRoutes(router: Router, routes: RouteDefinition[])`: Iterates over route array and registers handlers.
   - `handleRequestWithResponse(func: ServiceMethodWithResponse)`: Wraps service execution with automatic `next(error)` catch.
3. **Canonical Response Envelope & Utilities:**
   - `APIResponseEntity<T>`: Universal interface `{ status: boolean, statusCode: number, message: string, data: T }`.
   - `createSuccessResponse<T>`: Formats response data into the canonical envelope.
4. **Universal Error Handling & Middleware:**
   - `validateData(schema)`: Route-level Zod validation middleware returning standard 400 on failure.
   - `AppError`, `NotFoundError`, `UnauthorizedError`, `ValidationError`.
   - `correlationIdMiddleware`, `createErrorHandlerMiddleware`, and `createAuthMiddleware`.
5. **One-Line Documentation:**
   - Every function and middleware MUST include a clear 1-line explanatory comment.

### Prohibitions
- `shared-utils` MUST NOT contain domain business logic, Mongoose models, TypeORM entities, or concrete repositories.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Registers declarative route definitions to the Express router
export const registerRoutes = (router: Router, routes: RouteDefinition[]) => {
  routes.forEach(route => {
    const middlewares = route.middlewares || [];
    router[route.method](route.path, ...middlewares, route.handler);
  });
};

// Wraps controller action and propagates uncaught exceptions to error middleware
export const handleRequestWithResponse = (func: ServiceMethodWithResponse) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try { await func(req, res); } catch (error) { next(error); }
  };
};
```

---

## 6. Agent Output Checklist
- [ ] `packages/shared-utils/` created with standalone manifest.
- [ ] Enums defined for HTTP methods, status codes, common messages, environments, and log levels.
- [ ] `registerRoutes` and `handleRequestWithResponse` exported.
- [ ] 1-line comments on every utility function and middleware.
- [ ] Zero domain business logic or database entities in `shared-utils`.

---

## 7. Common Mistakes
- **Hardcoding String Literals in Services:** Writing raw strings instead of importing enums from `shared-utils`.
- **Tight Coupling to Monorepo Paths:** Using relative paths like `../../../packages` preventing extraction to an external repository.
