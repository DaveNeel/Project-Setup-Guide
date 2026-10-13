# Folder Structure: Shared Utils Blueprint

Literal file and directory blueprint for the shared library package (`packages/shared-utils`), structured for seamless extraction to an independent repository if required.

```
packages/shared-utils/
├── package.json                                 # Standalone manifest ("name": "shared-utils")
├── tsconfig.json                                # TSConfig extending root tsconfig.base.json
└── src/
    ├── index.ts                                 # Public export barrel
    ├── enums/                                   # Universal system enums (No static strings)
    │   ├── http-methods.enum.ts                 # HTTPRequestMethodsEnumEntity (GET, POST, etc.)
    │   ├── http-status-codes.enum.ts            # HttpStatusCodesEnumEntity (200, 201, 400, etc.)
    │   ├── common-messages.enum.ts              # CommonMessagesEnumEntity
    │   ├── node-env.enum.ts                     # NodeEnvEnum (DEVELOPMENT, PRODUCTION, TEST)
    │   ├── log-level.enum.ts                    # LogLevelEnum (DEBUG, INFO, WARN, ERROR)
    │   └── health-status.enum.ts                # HealthStatusEnum (OK, ERROR, UP, DOWN)
    ├── routes/
    │   ├── route-definition.interface.ts        # RouteDefinition interface
    │   ├── register-routes.util.ts              # registerRoutes(router, routes) with 1-line comment
    │   └── handle-request.util.ts               # handleRequestWithResponse wrapper
    ├── errors/
    │   ├── app.error.ts                         # Base abstract application error
    │   ├── not-found.error.ts
    │   ├── unauthorized.error.ts
    │   └── validation.error.ts
    ├── envelopes/
    │   ├── api-response.interface.ts            # APIResponseEntity<T> canonical envelope
    │   └── error-response.interface.ts          # Error envelope types
    ├── middleware/
    │   ├── validate-data.middleware.ts          # validateData(schema) route middleware
    │   ├── correlation-id.middleware.ts         # Correlation ID middleware with 1-line comment
    │   ├── error-handler.middleware.ts          # Central error handler with 1-line comment
    │   └── auth.middleware.ts                   # createAuthMiddleware with 1-line comment
    ├── logger/
    │   └── logger.ts                            # Structured Pino JSON logger factory
    └── utils/
        └── response.util.ts                     # createSuccessResponse helper returning APIResponseEntity
```
