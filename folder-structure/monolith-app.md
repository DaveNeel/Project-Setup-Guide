# Folder Structure: Monolith Application Blueprint

Literal file and directory blueprint for a unified Monolith or Modular Monolith application (pure source code baseline, no test folder).

```
<project-root>/
├── package.json                                 # Single unified application manifest
├── tsconfig.json                                # TypeScript configuration
├── .env.example
├── .gitignore
├── .prettierrc
├── eslint.config.mjs
└── src/
    ├── main.ts                                  # (or server.ts for Express)
    ├── app.module.ts                            # (or app.ts for Express)
    ├── config/
    │   ├── config.service.ts
    │   └── env.validation.ts                    # Startup validation schema
    ├── common/
    │   ├── errors/
    │   ├── filters/                             # (or middleware/ for Express)
    │   ├── interceptors/
    │   ├── logger/
    │   └── utils/
    ├── database/
    │   ├── connection.ts                        # (or database.module.ts)
    │   ├── data-source.ts
    │   └── migrations/
    ├── health/
    │   ├── health.controller.ts                 # Single GET /health dynamic DB probe
    │   └── health.module.ts
    └── modules/
        ├── auth/
        │   ├── index.ts                         # Public module facade (mandatory for modular monolith)
        │   ├── auth.module.ts
        │   ├── auth.controller.ts
        │   ├── auth.service.ts
        │   └── dto/
        └── <domain>/
            ├── index.ts                         # Public domain interface facade
            ├── <domain>.module.ts
            ├── <domain>.controller.ts
            ├── <domain>.service.ts
            ├── dto/
            ├── entities/                        # Private to this domain
            └── repositories/                    # Private to this domain
```
