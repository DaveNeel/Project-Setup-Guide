# Folder Structure: NestJS Service Blueprint

Literal file and directory blueprint for a NestJS service or application (pure source code baseline, no test folder).

```
<service-dir>/
├── package.json
├── tsconfig.json
├── tsconfig.build.json
├── nest-cli.json
├── .env.example
└── src/
    ├── main.ts                                      # Bootstrap entry point
    ├── app.module.ts                                # Root application module
    ├── config/
    │   ├── config.module.ts
    │   ├── config.service.ts
    │   └── env.validation.ts                        # Zod/class-validator startup schema
    ├── common/
    │   ├── filters/
    │   │   └── all-exceptions.filter.ts             # Standard error envelope mapping
    │   ├── interceptors/
    │   │   └── response-transform.interceptor.ts    # Standard success envelope mapping
    │   ├── middleware/
    │   │   └── correlation-id.middleware.ts         # Inbound correlation ID propagation
    │   └── logger/
    │       └── logger.service.ts                    # Structured JSON Pino logger
    ├── database/
    │   ├── database.module.ts
    │   ├── data-source.ts                           # Standalone DataSource / client config
    │   └── migrations/
    ├── health/
    │   ├── health.module.ts
    │   └── health.controller.ts                     # Single GET /health dynamic DB probe
    └── modules/
        └── <feature>/
            ├── <feature>.module.ts
            ├── <feature>.controller.ts
            ├── <feature>.service.ts
            ├── dto/
            │   ├── create-<feature>.dto.ts
            │   └── update-<feature>.dto.ts
            ├── entities/                            # (or schemas/ for Mongoose)
            │   └── <feature>.entity.ts
            └── repositories/
                ├── <feature>.repository.interface.ts
                └── <feature>.repository.ts
```
