# Step-Wise Intake Script

This document is the exact protocol the AI agent MUST follow when interviewing a human user to define a new backend project.

---

## Agent Execution Rules
1. **Target Directory & Shared Utils Path First:** Always prompt for the target project folder path. If microservices architecture is selected, also prompt for the `shared-utils` folder path before proceeding.
2. **Ask One Step at a Time:** Do not barrage the user with all questions simultaneously.
3. **Dynamic Filtering:** Present ONLY the options valid based on prior answers.
4. **Enforce Support Tiers:** Clearly label options with their Tier (`[Tier 1]`, `[Tier 2]`, `[Tier 3]`).
5. **No Test Folders in Baseline:** The generated project focuses purely on runnable source code (`src/`). Tests are not scaffolded in the baseline setup.
6. **Strict Enum-First Discipline:** The system replaces all magic strings and static response text with typed enums.
7. **Strict Separation of Router, Controller, and Service:**
   - **Router:** Declares route endpoints as `RouteDefinition[]` with route-level middleware `validateData(schema)` and registers them via `registerRoutes(router, routes)`.
   - **Controller:** Pure HTTP dispatch adapter. Extracts typed payload/params from `req`, calls domain service methods, envelopes service results via `createSuccessResponse`, and handles responses via `handleRequestWithResponse`. Controllers MUST NOT contain business logic.
   - **Service:** Contains pure business logic. Receives plain TypeScript data/DTOs and returns domain models. Services MUST NOT receive Express `req` or `res` objects, nor invoke `.safeParse()`.
8. **Shared Database & Centralized Migrations in `shared-utils`:** If microservices and a single shared database (`shared-db`) are chosen:
   - Database schemas, ORM entities, migrations, and the migration runner (`migration-runner.ts` supporting `migrate:status`, `migrate:run`, `migrate:generate`, `migrate:revert`) MUST reside and execute centrally inside `shared-utils`.
   - Microservices are STRICTLY PROHIBITED from defining migrations, executing migrations at boot, or containing migration CLI scripts in their local `package.json`.
   - The migration runner in `shared-utils` MUST support in-memory environment variable / CLI parameter injection (`DB_HOST`, `DB_PORT`, etc.) without writing temporary plaintext credential files (`temp-data-source.ts`) to disk.
   - `shared-utils` MUST NOT contain `.env` files, config loaders, or hardcoded database credentials on disk.
   - Database secrets MUST reside in each microservice's `.env` and be passed as typed parameters into the connection factory (`createDataSource(dbConfig)`).
9. **Pure TypeScript in `shared-utils` (No Map Files):** `shared-utils` MUST contain only clean `.ts` source files. Compilation map files like `.d.ts.map` and `.js.map` are strictly prohibited.
10. **Autonomous Execution After Approval:** Once human confirmation is granted on the generation plan, execute the entire scaffolding and verification pipeline autonomously without prompting for redundant intermediate permissions.
11. **Canonical API Response Entity:** All endpoints return `{ status: boolean, statusCode: number, message: string, data: T }` conforming to `APIResponseEntity<T>`.
12. **Confirmation Step:** Compile responses into `project-spec.yaml` and require human confirmation before drafting the generation plan.
13. **Independent Service Dependencies (No Hoisting):** In microservices, every microservice MUST maintain its own dedicated `node_modules/` directory and independent `package-lock.json`. The parent directory acts strictly as an orchestrator runner (`install:all`, `build:all`, `start:all`, `clean:all`, `reinstall:all`) without hoisting dependencies.

---

## Step 0: Target Project Folder Path & Shared Utils Path
- **Question 0A (Target Project Root):** "What folder path should the new backend project be generated in?"
  - **Default:** `../<project-name>` (an external directory outside the standards repository).
  - **Mandatory Validation:** The path MUST NOT point to the root of the standards repository.
- **Question 0B (Shared Utils Path - if Microservices):** "What folder path should the shared-utils package be located in?"
  - **Default:** `<project-root>/packages/shared-utils` (or a dedicated external directory e.g. `../shared-utils`).
  - **Packaging Rule:** In each microservice's `package.json`, `shared-utils` will use the local folder path for initial setup (e.g. `"file:..."`) and be structured with repository metadata so it can be replaced later by a git repo URL or npm artifact.

---

## Step 1: Architecture Topology
- **Question:** "What architectural topology will this backend project use?"
- **Options:**
  - `microservices` [Tier 1] *(Default)* — Independent deployable services in a monorepo workspace.
  - `monolith` [Tier 1] — Single unified deployable application.
  - `modular-monolith` [Tier 2] — Single deployable application with strictly enforced internal domain boundaries.
- **Branching Rule:**
  - If `monolith` or `modular-monolith`: Mark Step 5 (DB Strategy), Step 6 (Service Boundaries), and Step 7 (Inter-Service Comm) as **SKIPPED**. In Step 8, automatically select `monolith-common-folder`.
  - If `microservices`: Proceed to Step 2 and keep Steps 5, 6, 7 active.

---

## Step 2: Runtime, Language & Framework
- **Question:** "Which backend framework will be used on Node.js + TypeScript?"
- **Options:**
  - `express` [Tier 1] *(Default)* — Declarative router-controller architecture with `registerRoutes` and `handleRequestWithResponse`.
  - `nestjs` [Tier 1] — Modular architecture with dependency injection, decorators, and controllers.
- **Depends on:** Step 1.

---

## Step 3: Database Engine
- **Question:** "Which database engine will power the application?"
- **Options:**
  - `mongodb` [Tier 1] *(Default)* — Document-based NoSQL MongoDB.
  - `postgres` [Tier 1] — Relational PostgreSQL.
  - `mysql` [Tier 2] — Relational MySQL.
  - `mssql` [Tier 2] — Relational Microsoft SQL Server.
- **Depends on:** Step 2.
- **Branching Rule:**
  - If `mongodb`: Next step offers `mongoose` only.
  - If `postgres`, `mysql`, or `mssql`: Next step offers SQL data-access layers.

---

## Step 4: Data-Access Layer (ORM / ODM)
- **Question:** "Which data-access layer should be used for data persistence?"
- **Options (if MongoDB in Step 3):**
  - `mongoose` [Tier 1] *(Default & Only valid option)* — Schema-based ODM for MongoDB.
- **Options (if SQL in Step 3):**
  - `typeorm` [Tier 1] *(Default)* — Active Record & Data Mapper ORM.
  - `prisma` [Tier 2] — Schema-first type-safe ORM.
  - `drizzle` [Tier 2] — TypeScript-first lightweight SQL ORM.
- **Depends on:** Step 3.

---

## Step 5: Database Strategy (Microservices Only)
- **Condition:** Skipped if Step 1 is `monolith` or `modular-monolith`.
- **Question:** "What database segregation strategy will the microservices follow?"
- **Options:**
  - `db-per-service` [Tier 1] *(Default)* — Each service connects to its own dedicated database instance or catalog; no cross-service database access.
  - `shared-db` [Tier 1] — Services share a physical database cluster with strictly segregated schemas/collections.
- **Architectural Rule (if shared-db):**
  - Database schemas, ORM entities, migrations, and the migration runner (`migration-runner.ts` supporting `migrate:status`, `migrate:run`, `migrate:generate`, `migrate:revert`) MUST reside and execute centrally inside `shared-utils`.
  - Microservices are STRICTLY PROHIBITED from defining migrations, executing migrations at boot, or containing migration CLI scripts in their local `package.json`.
  - The migration runner in `shared-utils` MUST support in-memory environment variable / CLI parameter injection (`DB_HOST`, `DB_PORT`, etc.) without writing temporary plaintext credential files (`temp-data-source.ts`) to disk.
  - `shared-utils` MUST NOT contain `.env` files or hardcoded database credentials on disk.
  - Database secrets MUST reside in each microservice's `.env` and be passed as typed parameters into the connection factory.
- **Depends on:** Steps 1 and 3.

---

## Step 6: Service List & Boundaries (Microservices Only)
- **Condition:** Skipped if Step 1 is `monolith` or `modular-monolith`.
- **Question:** "Please specify the initial services: name, primary responsibility, and whether it owns a database (`own_db: true/false`)."
- **Naming Rule:** Names MUST be lowercase kebab-case ending with `-service`.
- **Default Example:**
  ```yaml
  services:
    - name: "user-service"
      responsibility: "User accounts, profile management, identity"
      own_db: true
      port: 3001
    - name: "notification-service"
      responsibility: "Email, SMS, and push notification dispatch"
      own_db: true
      port: 3002
    - name: "subscription-service"
      responsibility: "Subscription tiers, membership, and billing lifecycle"
      own_db: true
      port: 3003
  ```

---

## Step 7: Inter-Service Communication (Microservices Only)
- **Condition:** Skipped if Step 1 is `monolith` or `modular-monolith`.
- **Question:** "How will microservices communicate with one another?"
- **Options:**
  - `rest` [Tier 1] *(Default)* — Synchronous HTTP/REST with correlation ID propagation and standard envelopes.
  - `rabbitmq` [Tier 2] — Asynchronous event-driven messaging with event envelopes and DLQ.
  - `grpc` [Tier 3] — Protocol Buffers RPC *(Tier 3: NOT YET SUPPORTED)*.
- **Depends on:** Step 1.

---

## Step 8: Shared Code Strategy (`shared-utils`)
- **Question (if Microservices):** "How should common utilities, enums, errors, and route helpers be packaged?"
- **Options (Microservices):**
  - `shared-utils` [Tier 1] *(Default)* — Shared package named `shared-utils` containing enums, constants, route registrars, and error handlers (and shared database entities/connection factory if `shared-db` is selected). Structured as an independent package. In each microservice's `package.json`, use the local folder path (e.g. `"file:..."`) for initial setup, structured with repository metadata so it can be replaced later by a git repo URL or npm artifact.
  - `git-submodule` [Tier 2] — External repository tracked via Git submodules.
  - `published-private-packages` [Tier 2] — Private npm registry package.
  - `none` [Tier 3] — No shared code *(NOT YET SUPPORTED)*.
- **Monolith Rule:** Automatically resolved to `monolith-common-folder` [Tier 1] (`src/common/`).

---

## Step 9: Config & Environment Variables
- **Question:** "How should environment configuration and secrets be handled?"
- **Options:**
  - `typed-config` [Tier 1] *(Default)* — `.env` files with fail-fast Zod schema validation at boot. Enforces `NodeEnvEnum` and `LogLevelEnum`.
  - `aws-secrets-manager` [Tier 2] — Remote secret resolution via AWS Secrets Manager.
  - `vault` [Tier 3] — HashiCorp Vault *(NOT YET SUPPORTED)*.

---

## Step 10: Authentication
- **Question:** "What authentication mechanism should be implemented?"
- **Options:**
  - `jwt` [Tier 1] *(Default)* — Stateless JWT Bearer tokens with auth guards, claims validation, and user context extraction.
  - `session` [Tier 3] — Stateful cookie/session store *(NOT YET SUPPORTED)*.
  - `none` [Tier 3] — Public API without authentication guards *(NOT YET SUPPORTED)*.

---

## Free-Text Notes & Confirmation
- **Prompt:** "Any additional project notes, custom port requirements, or domain specifics?"
- **Confirmation:** Format all answers into `project-spec.yaml`, display it to the user, and prompt:
  > *"Please review and confirm this project specification. Once confirmed, I will generate the detailed generation plan."*
