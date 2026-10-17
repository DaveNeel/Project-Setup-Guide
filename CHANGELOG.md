# Changelog

All notable changes to the AI-Ready Project Standards repository are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-10-17

### Changed & Refined
- **Single Universal Entry Point (`AGENTS.md`)**:
  - Consolidated all AI agent instructions into `AGENTS.md` as the unified standard for all tools (Claude, Gemini, Antigravity, Cursor, OpenAI, etc.).
  - Removed redundant thin pointer files `CLAUDE.md` and `GEMINI.md`.
- **Elimination of Machine-Specific Folder Paths**:
  - Removed all hardcoded absolute local paths from documentation, markdown links, and example specifications.
  - Converted all documentation cross-references to clean, portable repository-relative links.
- **Microservices Shared-Utils & Git Repo Strategy**:
  - Added Step 0B prompt and schema properties (`shared_utils_path` and `shared_utils_repo_url`) for microservices.
  - Configured microservice `package.json` to link to `shared-utils` via relative local folder path (`"file:..."`) initially, with git repository metadata configured for seamless transition to git URL or private registry.
- **Shared Database Architecture & Zero Secrets in `shared-utils`**:
  - Centralized entity schemas, migrations, generic DB operations (`dbOperations.ts`), and parameterized connection factory `createDataSource(dbConfig)` in `shared-utils` when `shared-db` strategy is selected.
  - Strictly banned `.env` and hardcoded database credentials in `shared-utils`; database secrets reside exclusively in each microservice's `.env`.
- **Centralized Migration Suite in `shared-utils` (Approach 2 - In-Memory Parameter Injection)**:
  - Centralized schema generation, migration status checking, migration runs, and rollback commands (`migrate:status`, `migrate:run`, `migrate:generate`, `migrate:revert`) exclusively inside `shared-utils`.
  - Implemented dynamic in-memory parameter injection via `migration-runner.ts` and `migration-data-source.ts`, accepting CLI flags or environment variables with zero disk-based credential caching or temporary files.
  - Completely prohibited `migrations/` directories and migration execution scripts in microservices for shared databases, ensuring schema synchronization and data integrity remain unified.
- **Strict Architectural Layer Separation**:
  - Router: Pure route definition (`RouteDefinition[]`) and route middleware (`validateData(schema)`).
  - Controller: Pure HTTP dispatch adapter with zero domain business logic.
  - Service: Pure business logic accepting and returning plain domain data, strictly prohibited from accessing Express `req` or `res` objects.
- **Pure TypeScript in `shared-utils`**:
  - Enforced pure `.ts` source distribution; prohibited `.d.ts.map` and `.js.map` generation by setting `"sourceMap": false` and `"declarationMap": false`.
- **Independent Service Dependencies & Parent Orchestrator (No Hoisting)**:
  - Eliminated hoisted npm workspaces to prevent cross-service dependency bleeding, phantom imports, and broken containerization.
  - Mandated dedicated `node_modules/` folder and independent `package-lock.json` file inside every individual microservice.
  - Structured parent repository `package.json` purely as an orchestration manifest (`install:all`, `build:all`, `start:all`, `clean:all`, `reinstall:all`) using `concurrently` and `rimraf`.
- **Autonomous Post-Approval Execution**:
  - Formulated continuous execution pipeline after user approval to eliminate redundant interactive confirmations during scaffolding, installation, build, lint, and audit.

## [0.3.0] - 2026-10-10

### Changed & Refined
- **Route-Level Zod Validation Middleware (`validateData`)**:
  - Zod schemas are defined at the microservice module level (`*.dto.ts`).
  - Validation execution moved strictly into route `middlewares` via `validateData(schema)`.
  - Service methods prohibited from invoking `.safeParse()` or `.parse()`.
  - Validation failures return formatted HTTP 400 `APIResponseEntity` with first error issue message.
- **Unified `APIResponseEntity<T>` Envelope**:
  - Replaced legacy success/error envelopes with the standard:
    `{ status: boolean, statusCode: number, message: string, data: T }`.
  - Optimized generic typing (`<T = unknown>`) avoiding raw `any`.
  - Exported `createSuccessResponse` helper in `shared-utils` supporting dynamic status codes and messages.

## [0.2.0] - 2026-10-10

### Changed & Refined
- **Target Folder Path Prompt (Step 0)**:
  - Agent prompts for explicit target project folder path at the start of intake.
- **Shared Utils Architecture (`shared-utils`)**:
  - Renamed shared library to `shared-utils` with standalone repository structure (independent `package.json`, `tsconfig.json`, `src/`).
- **Strict Enum-First Discipline**:
  - Replaced all magic strings and static messages across the system with enums:
    - `HTTPRequestMethodsEnumEntity` for HTTP verbs.
    - `HttpStatusCodesEnumEntity` for status codes.
    - `CommonMessagesEnumEntity` for response messages.
    - `NodeEnvEnum`, `LogLevelEnum`, `HealthStatusEnum`.
- **Declarative Express Routing & Functional Controller Dispatch**:
  - Express routes declared via `RouteDefinition[]` and registered with `registerRoutes(router, routes)`.
  - Express controllers wrap service methods with `handleRequestWithResponse((req, res) => service.method(req, res))`.
- **Single Dynamic Health Check Probe**:
  - Replaced dual `/health/live` and `/health/ready` probes with a single `GET /health` endpoint performing active database connectivity verification.
- **Mandatory One-Line Documentation**:
  - Every reusable function, middleware, utility, and handler requires a concise 1-line explanatory comment.
- **Baseline Project Structure**:
  - Baseline project setup omits `test/` folders to prioritize clean, runnable source code (`src/`), verified via `npm run build` and `npm run lint`.
- **Commands & Tooling**:
  - Added `.claude/commands/create-project.md` alias alongside `new-project.md`.

## [0.1.0] - 2026-10-09

### Added
- **Canonical Routing & Intake Engine**:
  - `AGENTS.md` canonical agent router and rules of engagement.
  - Thin pointer entry points `CLAUDE.md` and `GEMINI.md`.
  - 11-step interactive intake script (`intake/step-wise-intake.md`).
  - Formal project specification schema (`intake/project-spec.schema.yaml`).
  - Commented YAML template and realistic example specification.
  - Decision graph (`intake/decision-tree.md`) and validation matrix (`intake/compatibility-matrix.md`).
- **Core Architectural Standards**:
  - Step 1: Architecture patterns (monolith, modular-monolith, microservices).
  - Step 2: Runtimes & frameworks (Node.js + TypeScript, NestJS, Express).
  - Step 3: Database engines (PostgreSQL, MongoDB, MySQL, MSSQL).
  - Step 4: Data-access layer & repository boundary (TypeORM, Mongoose, Prisma, Drizzle).
  - Step 5: Database strategy for microservices (db-per-service, shared-db).
  - Step 6: Service boundaries & mandatory service components.
  - Step 7: Inter-service communication (REST, RabbitMQ, gRPC).
  - Step 8: Shared code strategy (workspace packages, monolith common folder).
  - Step 9: Config & environment variables (fail-fast typed configuration).
  - Step 10: Authentication contracts (JWT Bearer tokens).
  - Step 11: Quality baseline (testing runner, ESLint/Prettier, strict TypeScript, code style).
  - Cross-cutting contracts (error envelope, structured logging, REST conventions, validation, health checks).
- **Literal Folder Structure Blueprints**:
  - Monorepo root, NestJS service, Express service, Monolith app, Shared workspace package.
- **Agent Workflows & Quality Checklists**:
  - `workflows/new-project.md`, `workflows/add-service.md`, `workflows/add-database.md`, `workflows/definition-of-done.md`.
  - Binary verification checklist `checklists/conformance-checklist.md`.
  - Human review checklist `checklists/human-review-checklist.md`.
- **Architectural Decision Records**:
  - ADR-001 through ADR-005.
- **Claude Code Integration**:
  - `.claude/skills/new-project/SKILL.md` and `.claude/commands/new-project.md`.
