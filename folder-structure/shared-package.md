# Folder Structure: Shared Utils Blueprint

Literal file and directory blueprint for the shared library package (`packages/shared-utils`), structured for seamless extraction to an independent repository if required.

```
packages/shared-utils/
├── package.json                                 # Standalone manifest ("name": "shared-utils", git repo metadata)
├── tsconfig.json                                # TSConfig (sourceMap: false, declarationMap: false)
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
    ├── interfaces/
    │   └── db.interface.ts                      # DBConfig / MSSQLConfig parameter interface
    ├── database/                                # Shared Database (when shared-db is selected)
    │   ├── entities/                            # All microservice shared DB entities
    │   │   ├── index.ts
    │   │   └── *.entity.ts
    │   ├── migrations/                          # Shared TypeORM migrations
    │   ├── migration-runner.ts                  # In-memory migration CLI runner (status, run, generate, revert)
    │   ├── migration-data-source.ts             # In-memory DataSource for migration runner
    │   ├── data-source.ts                       # createDataSource(dbConfig) runtime connection factory
    │   └── dbOperations.ts                      # Common DB CRUD & transaction helper utilities
    ├── logger/
    │   └── logger.ts                            # Structured Pino JSON logger factory
    └── utils/
        └── response.util.ts                     # createSuccessResponse helper returning APIResponseEntity
```
