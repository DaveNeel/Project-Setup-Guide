# Folder Structure: Express Service Blueprint

Literal file and directory blueprint for an Express microservice (pure source code baseline, no test folder).

```
<service-dir>/
├── package.json                                         # Service manifest (shared-utils: "file:..." with git URL structure)
├── tsconfig.json
├── .env.example                                         # Service secrets (DB credentials, PORT, JWT_SECRET)
└── src/
    ├── server.ts                                        # HTTP port listen & graceful shutdown
    ├── app.ts                                           # Express instance & middleware assembly
    ├── config/
    │   ├── env.config.ts                                # Validated config object with NodeEnvEnum & DB credentials
    │   └── validate-env.ts                              # Fail-fast startup validation schema
    ├── database/
    │   └── connection.ts                                # Boots DB via createDataSource(envConfig) from shared-utils (zero migrations here; managed in shared-utils)
    ├── health/
    │   ├── health.router.ts                             # Single GET /health router
    │   └── health.controller.ts                         # Dynamic DB connectivity probe
    └── modules/
        └── <feature>/
            ├── <feature>.router.ts                      # RouteDefinition[] with validateData & registerRoutes
            ├── <feature>.controller.ts                  # Pure HTTP adapter wrapped via handleRequestWithResponse (zero business logic)
            ├── <feature>.service.ts                     # Pure business logic implementation (receives DTOs, no req/res, no .safeParse)
            ├── dto/
            │   ├── create-<feature>.dto.ts              # Zod validation schema & inferred DTO
            │   └── update-<feature>.dto.ts
            ├── entities/
            │   └── <feature>.domain.ts                  # Plain domain interface (ORM entities imported from shared-utils in shared-db)
            └── repositories/
                ├── <feature>.repository.interface.ts    # Mandatory repository boundary
                └── <feature>.repository.ts              # Repository using shared-utils entities & createDataSource
```
