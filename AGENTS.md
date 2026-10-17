# AGENTS.md — Universal AI Agent Instructions & Routing Table

This is the canonical, single entry point for all AI coding agents (Claude Code, Gemini CLI, Antigravity, Cursor, OpenAI, Copilot, etc.).

You are an AI code generator. This repository contains architectural standards, contracts, and an intake engine. It contains **no application code**. Your job is to intake choices step-by-step and generate a new backend project in a separate directory matching these contracts.

---

## 1. Hard Rules of Engagement

1. **Target Directory & Shared Utils Path:** Always prompt the user for the target project folder path at the start of intake. If microservices architecture is selected, also prompt for the `shared-utils` folder path. Never generate project code inside this repository.
2. **Step-Wise Intake:** Execute steps in order (1 to 10). Only offer options valid for previous answers.
3. **Never Guess Silently:** If an intake field is missing or ambiguous, ask. Never invent decisions.
4. **Shared Code Strategy (`shared-utils`) & Git Repo Path:** In microservices, common utilities MUST be isolated in `shared-utils` (structured as an independent package). In each microservice's `package.json`, specify `shared-utils` via local folder path for initial setup (e.g. `"file:..."`), structured with repository metadata so it can seamlessly transition to a git repository URL or npm artifact.
5. **Shared Database & Centralized Migrations in `shared-utils` (Zero Migrations in Microservices):** If microservices and a single shared database (`shared-db`) are chosen:
   - Database schemas, ORM entities, migrations, database utilities (`dbOperations`), and the migration runner (`migration-runner.ts` supporting `migrate:status`, `migrate:run`, `migrate:generate`, `migrate:revert`) MUST reside and execute centrally inside `shared-utils`.
   - Microservices are STRICTLY PROHIBITED from defining migrations, executing migrations at boot, or containing migration CLI scripts in their local `package.json`.
   - The migration runner in `shared-utils` MUST support in-memory environment variable / CLI parameter injection (`DB_HOST`, `DB_PORT`, etc.) without writing temporary plaintext credential files (`temp-data-source.ts`) to disk.
   - `shared-utils` MUST NOT contain `.env` files, config loaders, or hardcoded database credentials on disk.
   - Database secrets MUST reside in each microservice's `.env` and be passed as typed parameters into the connection factory (`createDataSource(dbConfig)`).
6. **Strict Separation of Router, Controller, and Service:**
   - **Router:** Declares route endpoints as `RouteDefinition[]` with route-level middleware `validateData(schema)` and registers them via `registerRoutes(router, routes)`.
   - **Controller:** Pure HTTP dispatch adapter. Extracts parameters/body from `req`, delegates to domain service methods, envelopes service return values via `createSuccessResponse`, and returns HTTP responses via `handleRequestWithResponse`. Controllers MUST NOT contain domain business logic.
   - **Service:** Contains pure business logic. Service methods MUST accept plain data/DTOs and return domain models. Services MUST NOT receive Express `req` or `res` objects, nor invoke `.safeParse()`.
7. **Strict Enum-First Rule:** Prohibit magic strings, hardcoded codes, and static messages across the entire codebase. Use enums for HTTP methods, status codes, response messages, environments, log levels, and error codes.
8. **Single Health Probe:** Services MUST expose exactly one health check endpoint (`GET /health`) that actively verifies database connectivity, returning dynamic status.
9. **One-Line Documentation:** Every reusable function, middleware, utility, and handler MUST include a clear 1-line comment explaining its purpose.
10. **Basic Structure (No Test Folders):** The initial project setup scaffolds pure runnable source code (`src/`). Do NOT generate `test/` folders in the baseline scaffold.
11. **Pure TypeScript in `shared-utils` (No Map Files):** `shared-utils` MUST contain only clean `.ts` source files. Compilation map files like `.d.ts.map` and `.js.map` are strictly prohibited.
12. **Verify ORM Compatibility:** Check the ORM's current official documentation at generation time. Do not assume compatibility from memory.
13. **Approval Before Writing & Autonomous Execution:** Present the populated `project-spec.yaml` and generation plan. Wait for human approval before creating files. Once approved, execute the full scaffolding, installation, build, lint, and verification pipeline autonomously without prompting for redundant intermediate permissions.
14. **Route-Level Zod Validation Middleware:** Zod schemas are declared at the service/module level but validated exclusively via `validateData(schema)` in the route middleware array. Service methods MUST NOT call `.safeParse()` or `.parse()`.
15. **Standard API Response Entity (`APIResponseEntity<T>`):** All HTTP responses across the system MUST strictly adhere to the `{ status: boolean, statusCode: number, message: string, data: T }` envelope.
16. **Verify Runnable Build:** Run `npm run build` and `npm run lint` in the target project. Fix all failures before completing.
17. **Conformance Audit:** Evaluate output against `checklists/conformance-checklist.md` item-by-item and output the completed audit.
18. **Independent Service Dependencies & Parent Orchestrator (No Hoisted Workspaces):** In microservices, every microservice MUST maintain its own dedicated `node_modules/` directory and independent `package-lock.json` file for autonomous execution and Dockerization. Hoisting dependencies to a shared root `node_modules` is prohibited. The parent repository root `package.json` acts strictly as an orchestration manifest providing lifecycle scripts (`install:all`, `build:all`, `start:all`, `clean:all`, `reinstall:all`) using `concurrently` and `rimraf` to execute commands inside each service.

---

## 2. Standards Routing Table

| Step | User Choice | Files to Load |
|:---|:---|:---|
| **Intake** | (All projects) | `intake/step-wise-intake.md`, `intake/decision-tree.md`, `intake/compatibility-matrix.md` |
| **01 Architecture** | `microservices`<br>`monolith`<br>`modular-monolith` | `standards/01-architecture/microservices.md`<br>`standards/01-architecture/monolith.md`<br>`standards/01-architecture/modular-monolith.md`, `standards/01-architecture/monolith.md` |
| **02 Framework** | `express`<br>`nestjs` | `standards/02-framework/express.md`<br>`standards/02-framework/nestjs.md` |
| **03 Database** | `mongodb`<br>`postgres`<br>`mysql`<br>`mssql` | `standards/03-database/nosql-common.md`, `standards/03-database/engines/mongodb.md`<br>`standards/03-database/relational-common.md`, `standards/03-database/engines/postgres.md`<br>`standards/03-database/relational-common.md`, `standards/03-database/engines/mysql.md`<br>`standards/03-database/relational-common.md`, `standards/03-database/engines/mssql.md` |
| **04 Data Access** | `mongoose`<br>`typeorm`<br>`prisma`<br>`drizzle` | `standards/04-data-access/repository-pattern.md`, `standards/04-data-access/mongoose.md`<br>`standards/04-data-access/repository-pattern.md`, `standards/04-data-access/typeorm.md`<br>`standards/04-data-access/repository-pattern.md`, `standards/04-data-access/prisma.md`<br>`standards/04-data-access/repository-pattern.md`, `standards/04-data-access/drizzle.md` |
| **05 DB Strategy** | `db-per-service`<br>`shared-db`<br>(Monolith) | `standards/05-db-strategy/db-per-service.md`<br>`standards/05-db-strategy/shared-db.md`<br>*(Skipped)* |
| **06 Services** | (Microservices)<br>(Monolith) | `standards/06-services/service-boundaries.md`, `standards/06-services/what-every-service-contains.md`<br>*(Skipped)* |
| **07 Inter-Comm** | `rest`<br>`rabbitmq`<br>`grpc` | `standards/07-communication/rest.md`<br>`standards/07-communication/rabbitmq.md`<br>`standards/07-communication/grpc.md` *(Tier 3 — Stop)* |
| **08 Shared Code** | `shared-utils`<br>`monolith-common-folder`<br>`git-submodule`<br>`published-private-packages` | `standards/08-shared-code/workspace-packages.md`<br>`standards/08-shared-code/monolith-common-folder.md`<br>`standards/08-shared-code/alternatives.md`<br>`standards/08-shared-code/alternatives.md` |
| **09 Config & Env** | `typed-config` + `env-naming`<br>`aws-secrets-manager`<br>`vault` | `standards/09-config-secrets/typed-config.md`, `standards/09-config-secrets/env-naming.md`<br>`standards/09-config-secrets/secrets-management.md`<br>`standards/09-config-secrets/secrets-management.md` *(Tier 3 — Stop)* |
| **10 Auth** | `jwt`<br>`session` / `none` | `standards/10-auth/jwt.md`<br>`standards/10-auth/session.md` or `standards/10-auth/none.md` *(Tier 3 — Stop)* |
| **11 Quality** | (All projects) | `standards/11-quality/lint-format.md`, `standards/11-quality/code-style.md`, `standards/11-quality/testing.md` |
| **Cross-Cutting** | (All projects) | `standards/cross-cutting/error-handling.md`, `standards/cross-cutting/logging.md`, `standards/cross-cutting/api-conventions.md`, `standards/cross-cutting/validation.md`, `standards/cross-cutting/health-checks.md` |
| **Layout Tree** | Microservices<br>Monolith<br>Express Service<br>NestJS Service<br>Shared Utils | `folder-structure/monorepo-root.md`<br>`folder-structure/monolith-app.md`<br>`folder-structure/express-service.md`<br>`folder-structure/nestjs-service.md`<br>`folder-structure/shared-package.md` |
| **Workflow** | (All projects) | `workflows/new-project.md`, `workflows/definition-of-done.md`, `checklists/conformance-checklist.md` |

---

## 3. Execution Sequence

1. Prompt for target project directory path.
2. Read `intake/step-wise-intake.md` and present questions Step 1 through 10.
3. Ingest answers into `project-spec.yaml` and validate against `intake/compatibility-matrix.md`.
4. Load matched documents via the Routing Table above.
5. Output generation plan and wait for user approval.
6. Follow `workflows/new-project.md` to scaffold, compile, lint, and audit.
