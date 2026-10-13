# Changelog

All notable changes to the AI-Ready Project Standards repository are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
