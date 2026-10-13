# Quality Standard: Code Style, Enums & Declarative Patterns (Tier 1)

## 1. Purpose
Defines coding style, 1-line documentation requirements, strict enum-everywhere conventions, and declarative routing patterns.

---

## 2. Applies when
- All projects across all architectural topologies.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Routing Pattern | Uses `registerRoutes` with `RouteDefinition[]`. Procedural router calls prohibited. |
| Controller Pattern| Uses `handleRequestWithResponse((req, res) => service.method(req, res))`. |

---

## 4. Contract

### Strict Enum-Everywhere Rule
- Raw string literals and magic numbers are STRICTLY PROHIBITED in business logic, routes, and responses:
  - HTTP verbs MUST use `HTTPRequestMethodsEnumEntity`.
  - Status codes MUST use `HttpStatusCodesEnumEntity`.
  - Common messages MUST use `CommonMessagesEnumEntity`.
  - Environments MUST use `NodeEnvEnum`.
  - Log levels MUST use `LogLevelEnum`.
  - Health statuses MUST use `HealthStatusEnum`.

### Mandatory One-Line Documentation
- Every function, middleware, utility, and handler MUST include a concise 1-line comment directly above it explaining what it does.

### Declarative Route & Controller Patterns
- Routes MUST be declared via `RouteDefinition[]`:
  ```typescript
  // Registers user management routes to the router instance
  const routes: RouteDefinition[] = [
    {
      path: UserRoutesEnum.CREATE_USER,
      method: HTTPRequestMethodsEnumEntity.POST,
      middlewares: [authMiddleware],
      handler: ctrl.createUser
    }
  ];
  registerRoutes(router, routes);
  ```
- Controller actions MUST delegate to services via `handleRequestWithResponse`:
  ```typescript
  // Dispatches user creation request to UserService
  public createUser = handleRequestWithResponse((req, res) => this.userService.createUser(req, res));
  ```

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Dispatches chat prompt to LLM service
public v2ChatWithLLM = handleRequestWithResponse((req, res) => this.llmService.v2ChatWithLLM(req, res));

  // Defines chat completion route definition
  const routes: RouteDefinition[] = [
    {
      path: ChatRoutesEnum.CHAT,
      method: HTTPRequestMethodsEnumEntity.POST,
      middlewares: [validateData(chatSchema), verifyTokenMiddleware, loggerMiddleware],
      handler: ctrl.v2ChatWithLLM
    }
  ];
  registerRoutes(router, routes);
```

---

## 6. Agent Output Checklist
- [ ] 1-line comments on every reusable function, middleware, and handler.
- [ ] Zero static string literals for HTTP methods, codes, environments, or status strings.
- [ ] Routes registered using `registerRoutes` with `RouteDefinition[]`.
- [ ] Controllers using `handleRequestWithResponse`.

---

## 7. Common Mistakes
- **Omitting Comments:** Creating utility functions or middlewares without a 1-line description.
- **Using Magic Strings:** Writing `res.status(200).send('Success')` instead of enums.
- **Writing Procedural Controllers:** Writing manual `async (req, res, next) => { try ... }` everywhere.
