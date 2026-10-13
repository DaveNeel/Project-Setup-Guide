# Folder Structure: Microservices Monorepo Root

Literal file and directory blueprint for a multi-service backend workspace.

```
<project-root>/
├── package.json                 # Workspace root manifest (workspaces: ["packages/*", "services/*"])
├── tsconfig.base.json           # Shared root TypeScript compiler options
├── .gitignore                   # Standard Node / environment exclusions
├── .prettierrc                  # Workspace formatting configuration
├── eslint.config.mjs            # Root ESLint flat configuration
├── README.md                    # Project documentation & setup instructions
├── AGENTS.md                    # Project-level agent instructions
├── packages/
│   └── shared-utils/            # Shared utilities, enums, route handlers (extractable to separate repo)
│       ├── package.json
│       ├── tsconfig.json
│       └── src/
└── services/
    ├── <service-1>/             # Autonomous microservice 1 (pure src/, no test/ folder)
    │   ├── package.json
    │   ├── tsconfig.json
    │   ├── .env.example
    │   └── src/
    └── <service-2>/             # Autonomous microservice 2
        ├── package.json
        ├── tsconfig.json
        ├── .env.example
        └── src/
```
