# Workflow: New Project Setup

This document defines the canonical sequential procedure an AI agent MUST execute to generate a backend project.

---

## 1. Target Directory Prompt & Intake
1. **Target Directory & Shared Utils Path:** Prompt the user for the target folder path where the code should be generated (e.g. `../my-backend-app`). If microservices architecture is selected, also prompt for the folder path where `shared-utils` should be located.
2. Read `AGENTS.md` to load rules of engagement.
3. Run the interview script in `intake/step-wise-intake.md` step-by-step.
4. Compile user responses into a concrete in-memory specification object.

---

## 2. Specification Validation
1. Validate specification against `intake/project-spec.schema.yaml`.
2. Check decision validity against `intake/decision-tree.md` and `intake/compatibility-matrix.md`.
3. If any choice is Tier 3, output the `[NOT YET SUPPORTED]` banner and halt.

---

## 3. Human Confirmation Gate
1. Display the fully populated `project-spec.yaml` to the user.
2. Request explicit confirmation before proceeding:
   > *"Please review and confirm this project specification. Once confirmed, I will draft the detailed generation plan."*
3. Wait for human confirmation.

---

## 4. Selective Standards Loading
1. Consult the **Standards Routing Table** in `AGENTS.md`.
2. Load ONLY the standards matching confirmed answers.
3. Verify chosen ORM/driver compatibility against current official package documentation.

---

## 5. Generation Plan & Human Approval Gate
1. Draft a comprehensive generation plan containing:
   - Target directory path and `shared-utils` folder path.
   - Complete directory and file tree (confirming no `test/` folders in baseline, and pure `.ts` in `shared-utils`).
   - Service port assignments and environment variable keys (with zero secrets in `shared-utils`).
   - Confirmation of `shared-utils` with enums, declarative route patterns, and parameterized DB connection.
   - Confirmation of strict Router -> Controller -> Service separation.
2. Present the plan to the user and **WAIT for explicit human approval** before creating files.
3. **Autonomous Non-Interactive Execution:** Once the user approves the generation plan, execute the remaining scaffolding, installation, build, lint, format, and audit steps autonomously without pausing to request redundant intermediate permissions.

---

## 6. Target Directory Scaffolding
1. Scaffold root workspace in the confirmed target directory.
2. Generate base workspace configurations from `folder-structure/`:
   - Root `package.json` (parent orchestrator with scripts: `install:all`, `build:all`, `clean:all`, `reinstall:all`, `migrate:status`, `migrate:run`), `tsconfig.base.json`, `eslint.config.mjs`, `.prettierrc`, `.gitignore`.

---

## 7. Layered Generation Sequence
Generate code files strictly in dependency layer order:
1. **Shared Utils Package (`packages/shared-utils/` or external path):**
   - Universal Enums: `HTTPRequestMethodsEnumEntity`, `HttpStatusCodesEnumEntity`, `CommonMessagesEnumEntity`, `NodeEnvEnum`, `LogLevelEnum`, `HealthStatusEnum`.
   - Route Helpers: `registerRoutes(router, routes)` and `handleRequestWithResponse(func)`.
   - Canonical Envelope: `APIResponseEntity<T>` and `createSuccessResponse<T>`.
   - Middleware: `validateData(schema)`, `correlationIdMiddleware`, `createErrorHandlerMiddleware`, `createAuthMiddleware`.
   - Error classes: `AppError`, `NotFoundError`, `UnauthorizedError`, `ValidationError`.
   - Shared Database (when `shared-db` is selected):
     - All ORM entities/schemas (`entities/`), generic operations (`dbOperations.ts`), and parameterized connection factory `createDataSource(dbConfig)` in `data-source.ts`. Zero secrets in `shared-utils`.
     - Centralized Database Migration Suite: `migration-runner.ts`, `migration-data-source.ts`, `migrations/` directory, and migration lifecycle scripts (`migrate:status`, `migrate:run`, `migrate:generate`, `migrate:revert`). Utilizes Approach 2 (in-memory parameter injection from CLI flags / env vars with zero filesystem credentials creation).
   - Pure TypeScript Baseline: clean `.ts` files only; `.d.ts.map` and `.js.map` map files are prohibited.
   - *Rule:* 1-line comments on all functions and middlewares.
2. **Configuration & Database in Services:**
   - Fail-fast typed configuration validating local service secrets (`PORT`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `JWT_SECRET`).
   - Database connection management boots via `createDataSource(envConfig.db)` imported from `shared-utils` passing parameters.
   - *Rule:* Microservices contain zero migration directories and zero migration execution scripts. Schema synchronization and migrations are strictly isolated to `shared-utils`.
3. **Single Dynamic Health Check:**
   - Mount `GET /health` executing active DB connection probe (`SELECT 1` or `readyState === 1`).
4. **Domain Modules (Strict Layer Separation):**
   - Router: declares `RouteDefinition[]` with `validateData(schema)` in `middlewares` and mounted via `registerRoutes`.
   - Controller: pure HTTP adapter using `handleRequestWithResponse`, receives `(req, res)`, delegates to service, envelopes results via `createSuccessResponse`, and returns HTTP response. Zero business logic in controllers.
   - Domain Service: pure business logic implementation (receives plain data/DTOs, returns domain models, zero Express `req`/`res` dependencies, zero `.safeParse()` calls).
   - Repository: interface & concrete repository implementing data access.
   - *Rule:* Zero `test/` folders in baseline setup.

---

## 8. Support Artifacts & Provenance Stamping
1. Generate `.env.example` in each service root documenting required keys.
2. Stamp root `README.md` with: `Scaffolded using AI-Ready Project Standards v0.4.0`.
3. Generate project-level `AGENTS.md` in target repository.

---

## 9. Verification & Execution Loop
Execute real commands in the target project directory:
1. `npm install` inside each individual microservice directory (or via root `npm run install:all`), ensuring each service maintains its own dedicated `node_modules/` folder and independent `package-lock.json` file.
2. `npm run build` — Verify zero TypeScript compilation errors across each service and root `build:all`.
3. `npm run lint` — Verify zero ESLint errors across each service and root `lint:all`.
4. *Rule:* Never claim a command passed without actually executing it.

---

## 10. Conformance Audit & Handoff
1. Evaluate generated project against `checklists/conformance-checklist.md`.
2. Output final report with audit results and execution log.
