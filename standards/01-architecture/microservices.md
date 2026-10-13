# Architecture Standard: Microservices (Tier 1)

## 1. Purpose
Defines the architectural contract for scaffolding a multi-service backend within a monorepo workspace.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Microservices |
|:---|:---|
| Step 2 (Framework) | Every service in `services/*` MUST use the framework chosen in Step 2. |
| Step 5 (DB Strategy) | Determines whether each service uses an isolated database (`db-per-service`) or shared instance. |
| Step 8 (Shared Code)| Establishes internal workspace packages under `packages/*` for shared types/middleware. |

---

## 4. Contract

### Repository & Workspace Structure
- The repository root MUST be organized as a workspace monorepo managing multiple service packages.
- All microservices MUST reside in the `services/` directory: `services/<service-name>/`.
- Each service MUST have its own independent `package.json`, `tsconfig.json`, and source tree (`src/`).
- Each service MUST be independently buildable and verifiable via its local scripts:
  - `npm run build`
  - `npm run lint`
- Each service MUST bind to a configurable, dedicated HTTP port defined via environment variable `PORT`.
- Services MUST NOT share mutable runtime memory, global state, or cross-import internal source files from other services.
- Shared utilities MUST be consumed strictly through `shared-utils` located in `packages/shared-utils` (or standalone repository) declared as a dependency in `package.json`.

---

## 5. Illustrative Tree
<!-- illustrative only -->
```
my-backend-monorepo/
├── package.json              # Monorepo workspace root manifest
├── pnpm-workspace.yaml       # Workspace package definitions
├── packages/
│   └── shared-utils/         # Shared utilities, enums, route handlers
└── services/
    ├── auth-service/         # Autonomous service 1
    └── core-service/         # Autonomous service 2
```

---

## 6. Agent Output Checklist
- [ ] Root `package.json` contains `workspaces` array including `"services/*"` and `"packages/*"`.
- [ ] Each service in `services/` contains isolated `package.json`, `tsconfig.json`, and `src/`.
- [ ] Service ports are assigned distinct defaults (e.g. 3001, 3002).
- [ ] Zero relative imports between sibling services (`../../services/...` is prohibited).

---

## 7. Common Mistakes
- **Cross-Service File Imports:** Importing a TypeScript file from one service into another (`import ... from '../../service-b/src'`).
- **Unified Single Process Boot:** Writing a single `index.ts` at the root that boots all microservices in one process.
- **Port Collisions:** Hardcoding the same fallback port (e.g., 3000) across multiple services.
