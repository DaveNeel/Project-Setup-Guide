# Folder Structure: Express Service Blueprint

Literal file and directory blueprint for an Express microservice (pure source code baseline, no test folder).

```
<service-dir>/
├── package.json
├── tsconfig.json
├── .env.example
└── src/
    ├── server.ts                                    # HTTP port listen & graceful shutdown
    ├── app.ts                                       # Express instance & middleware assembly
    ├── config/
    │   ├── env.config.ts                            # Validated config object with NodeEnvEnum
    │   └── validate-env.ts                          # Startup validation schema
    ├── database/
    │   ├── connection.ts                            # Database lifecycle & readyState health check
    │   └── migrations/
    ├── health/
    │   ├── health.router.ts                         # Single GET /health router
    │   └── health.controller.ts                     # Dynamic DB connectivity probe
    └── modules/
        └── <feature>/
            ├── <feature>.router.ts                  # RouteDefinition[] with validateData & registerRoutes
            ├── <feature>.controller.ts              # Handlers wrapped via handleRequestWithResponse
            ├── <feature>.service.ts                 # Pure business logic implementation (no .safeParse)
            ├── dto/
            │   ├── create-<feature>.dto.ts
            │   └── update-<feature>.dto.ts
            ├── models/                              # Mongoose schemas/models or SQL entities
            │   ├── <feature>.entity.ts              # Plain domain interface
            │   └── <feature>.model.ts
            └── repositories/
                ├── <feature>.repository.interface.ts # Mandatory repository boundary
                └── <feature>.repository.ts
```
