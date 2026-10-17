# Folder Structure: Microservices Monorepo Root

Literal file and directory blueprint for a multi-service backend workspace.

```
<project-root>/
├── package.json                 # Parent orchestrator manifest (scripts: install:all, build:all, migrate:status, migrate:run, clean:all)
├── package-lock.json            # Root lockfile for orchestrator tooling (concurrently, rimraf)
├── node_modules/                # Root orchestrator tools
├── tsconfig.base.json           # Shared root TypeScript compiler options
├── .gitignore                   # Standard Node / environment exclusions
├── .prettierrc                  # Workspace formatting configuration
├── eslint.config.mjs            # Root ESLint flat configuration
├── README.md                    # Project documentation & setup instructions
├── AGENTS.md                    # Project-level agent instructions
├── packages/
│   └── shared-utils/            # Shared utilities, enums, route handlers (extractable to separate repo)
│       ├── package.json
│       ├── package-lock.json
│       ├── node_modules/
│       ├── tsconfig.json
│       └── src/
└── services/
    ├── <service-1>/             # Autonomous microservice 1 (independent node_modules & lockfile)
    │   ├── package.json         # Dedicated service manifest
    │   ├── package-lock.json    # Dedicated independent service lockfile
    │   ├── node_modules/        # Dedicated independent service dependencies
    │   ├── tsconfig.json
    │   ├── .env.example
    │   └── src/
    └── <service-2>/             # Autonomous microservice 2 (independent node_modules & lockfile)
        ├── package.json
        ├── package-lock.json
        ├── node_modules/
        ├── tsconfig.json
        ├── .env.example
        └── src/
```
