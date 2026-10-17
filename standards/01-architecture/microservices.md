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

### Parent Orchestrator & Independent Service Dependencies (No Hoisting)
- The repository root MUST be organized as an orchestration parent directory managing multiple microservices, NOT a hoisted workspace.
- Hoisting dependencies to a single shared root `node_modules` is STRICTLY PROHIBITED.
- Every microservice MUST maintain its own dedicated `node_modules/` folder and independent `package-lock.json` file to guarantee true deployment autonomy and containerization independence.
- The parent root `package.json` acts strictly as an orchestration manifest providing lifecycle scripts (`install:all`, `build:all`, `start:all`, `clean:all`, `reinstall:all`) using `concurrently` and `rimraf` to execute commands inside each service.
- All microservices MUST reside in the `services/` directory (or as autonomous service directories under the root parent): `<service-name>/`.
- Each service MUST have its own independent `package.json`, `package-lock.json`, `node_modules/`, `tsconfig.json`, and source tree (`src/`).
- Each service MUST be independently buildable and verifiable via its local scripts:
  - `npm run build`
  - `npm run lint`
- Each service MUST bind to a configurable, dedicated HTTP port defined via environment variable `PORT`.
- Services MUST NOT share mutable runtime memory, global state, or cross-import internal source files from other services.
- **Shared Utils Consumption & Git Repo Path:**
  - Common utilities MUST be consumed strictly through `shared-utils` located at the confirmed `shared_utils_path`.
  - In each microservice's `package.json`, the dependency on `shared-utils` MUST use the local folder path (e.g. `"file:..."`) for initial setup, with repository metadata structured for a git repository URL (e.g. `git+https://...`) or npm artifact once published.
- **Shared Database Centralization:**
  - When `db_strategy: shared-db` is used, database schemas, entities, migrations, generic DB operations, and the connection factory (`createDataSource(dbConfig)`) MUST reside centrally inside `shared-utils`.
  - Zero secrets in `shared-utils`: database credentials reside exclusively in each service's `.env` and are passed as arguments at connection time.
- **Pure TypeScript in `shared-utils`:**
  - `shared-utils` MUST contain only clean `.ts` source files. Compilation map files like `.d.ts.map` and `.js.map` are strictly prohibited.

---

## 5. Illustrative Tree
<!-- illustrative only -->
```
backend/
├── package.json              # Parent orchestration manifest (scripts: install:all, build:all, clean:all)
├── package-lock.json         # Root lockfile for orchestrator tooling (concurrently, rimraf)
├── node_modules/             # Root orchestrator tools
├── user-service/             # Autonomous service 1
│   ├── package.json          # Independent service manifest
│   ├── package-lock.json     # Dedicated independent service lockfile
│   ├── node_modules/         # Dedicated independent service dependencies
│   ├── tsconfig.json
│   ├── .env.example
│   └── src/
└── product-service/          # Autonomous service 2
    ├── package.json          # Independent service manifest
    ├── package-lock.json     # Dedicated independent service lockfile
    ├── node_modules/         # Dedicated independent service dependencies
    ├── tsconfig.json
    ├── .env.example
    └── src/
```

---

## 6. Agent Output Checklist
- [ ] Root `package.json` provides orchestration scripts (`install:all`, `build:all`, `clean:all`, `reinstall:all`) without hoisted `workspaces` array.
- [ ] Each microservice contains its own dedicated `node_modules/` folder and independent `package-lock.json`.
- [ ] Each service contains isolated `package.json`, `tsconfig.json`, and `src/`.
- [ ] Service ports are assigned distinct defaults (e.g. 3001, 3002).
- [ ] Zero relative imports between sibling services (`../../services/...` is prohibited).

---

## 7. Common Mistakes
- **Hoisting Dependencies with Workspaces:** Using `workspaces: []` to create a single shared root `node_modules` and single `package-lock.json`, preventing independent containerization and autonomous service execution.
- **Cross-Service File Imports:** Importing a TypeScript file from one service into another (`import ... from '../../service-b/src'`).
- **Unified Single Process Boot:** Writing a single `index.ts` at the root that boots all microservices in one process.
- **Port Collisions:** Hardcoding the same fallback port (e.g., 3000) across multiple services.
