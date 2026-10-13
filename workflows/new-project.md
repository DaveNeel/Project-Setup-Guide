# Workflow: New Project Setup

This document defines the canonical sequential procedure an AI agent MUST execute to generate a backend project.

---

## 1. Target Directory Prompt & Intake
1. **Target Directory First:** Prompt the user for the target folder path where the code should be generated (e.g. `../my-backend-app`).
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
   - Target directory path.
   - Complete directory and file tree (confirming no `test/` folders in baseline).
   - Service port assignments and environment variable keys.
   - Confirmation of `shared-utils` with enums and declarative route patterns.
2. Present the plan to the user and **WAIT for explicit human approval** before creating files.

---

## 6. Target Directory Scaffolding
1. Scaffold root workspace in the confirmed target directory.
2. Generate base workspace configurations from `folder-structure/`:
   - Root `package.json`, `tsconfig.base.json`, `eslint.config.mjs`, `.prettierrc`, `.gitignore`.

---

## 7. Layered Generation Sequence
Generate code files strictly in dependency layer order:
1. **Shared Utils Package (`packages/shared-utils/`):**
   - Universal Enums: `HTTPRequestMethodsEnumEntity`, `HttpStatusCodesEnumEntity`, `CommonMessagesEnumEntity`, `NodeEnvEnum`, `LogLevelEnum`, `HealthStatusEnum`.
   - Route Helpers: `registerRoutes(router, routes)` and `handleRequestWithResponse(func)`.
   - Canonical Envelope: `APIResponseEntity<T>` and `createSuccessResponse<T>`.
   - Middleware: `validateData(schema)`, `correlationIdMiddleware`, `createErrorHandlerMiddleware`, `createAuthMiddleware`.
   - Error classes: `AppError`, `NotFoundError`, `UnauthorizedError`, `ValidationError`.
   - *Rule:* 1-line comments on all functions and middlewares.
2. **Configuration & Database:**
   - Fail-fast typed configuration using enums.
   - Database connection management with `readyState` health check.
3. **Single Dynamic Health Check:**
   - Mount `GET /health` executing active DB connection probe.
4. **Domain Modules (Declarative Pattern):**
   - Repository interface & concrete Mongoose/TypeORM repository.
   - Domain Service containing pure business logic (no `.safeParse()` calls).
   - Controller using `handleRequestWithResponse((req, res) => service.method(req, res))`.
   - Router declaring `RouteDefinition[]` with `validateData(schema)` in `middlewares` and mounted via `registerRoutes`.
   - *Rule:* Zero `test/` folders in baseline setup.

---

## 8. Support Artifacts & Provenance Stamping
1. Generate `.env.example` in each service root documenting required keys.
2. Stamp root `README.md` with: `Scaffolded using AI-Ready Project Standards v0.3.0`.
3. Generate project-level `AGENTS.md` in target repository.

---

## 9. Verification & Execution Loop
Execute real commands in the target project directory:
1. `npm install`
2. `npm run build` — Verify zero TypeScript compilation errors.
3. `npm run lint` — Verify zero ESLint errors.
4. *Rule:* Never claim a command passed without actually executing it.

---

## 10. Conformance Audit & Handoff
1. Evaluate generated project against `checklists/conformance-checklist.md`.
2. Output final report with audit results and execution log.
