# AGENTS.md — Canonical AI Agent Instructions & Routing Table

You are an AI code generator. This repository contains architectural standards, contracts, and an intake engine. It contains **no application code**. Your job is to intake choices step-by-step and generate a new backend project in a separate directory matching these contracts.

---

## 1. Hard Rules of Engagement

1. **Target Directory First:** Always prompt the user for the target project folder path at the start of intake. Never generate project code inside this repository.
2. **Step-Wise Intake:** Execute steps in order (1 to 10). Only offer options valid for previous answers.
3. **Never Guess Silently:** If an intake field is missing or ambiguous, ask. Never invent decisions.
4. **Shared Code Strategy (`shared-utils`):** In microservices, common utilities MUST be isolated in `shared-utils` (structured so it can live as a monorepo workspace package or an independent git repository).
5. **Strict Enum-First Rule:** Prohibit magic strings, hardcoded codes, and static messages across the entire codebase. Use enums for HTTP methods, status codes, response messages, environments, log levels, and error codes.
6. **Declarative Routing & Controller Handlers:** Express projects MUST use `registerRoutes(router, routes)` with `RouteDefinition[]` and `handleRequestWithResponse((req, res) => service.method(req, res))`. Old procedural router-controller patterns are prohibited.
7. **Single Health Probe:** Services MUST expose exactly one health check endpoint (`GET /health`) that actively verifies database connectivity, returning dynamic status.
8. **One-Line Documentation:** Every reusable function, middleware, utility, and handler MUST include a clear 1-line comment explaining its purpose.
9. **Basic Structure (No Test Folders):** The initial project setup scaffolds pure runnable source code (`src/`). Do NOT generate `test/` folders in the baseline scaffold.
10. **Verify ORM Compatibility:** Check the ORM's current official documentation at generation time. Do not assume compatibility from memory.
11. **Approval Before Writing:** Present the populated `project-spec.yaml` and generation plan. Wait for human approval before creating files.
12. **Route-Level Zod Validation Middleware:** Zod schemas are declared at the service/module level but validated exclusively via `validateData(schema)` in the route middleware array. Service methods MUST NOT call `.safeParse()` or `.parse()`.
13. **Standard API Response Entity (`APIResponseEntity<T>`):** All HTTP responses across the system MUST strictly adhere to the `{ status: boolean, statusCode: number, message: string, data: T }` envelope.
14. **Verify Runnable Build:** Run `npm run build` and `npm run lint` in the target project. Fix all failures before completing.
15. **Conformance Audit:** Evaluate output against `checklists/conformance-checklist.md` item-by-item and output the completed audit.

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
