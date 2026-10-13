# Step-Wise Intake Script

This document is the exact protocol the AI agent MUST follow when interviewing a human user to define a new backend project.

---

## Agent Execution Rules
1. **Target Directory First:** Always prompt for the target project folder path before asking architecture questions.
2. **Ask One Step at a Time:** Do not barrage the user with all questions simultaneously.
3. **Dynamic Filtering:** Present ONLY the options valid based on prior answers.
4. **Enforce Support Tiers:** Clearly label options with their Tier (`[Tier 1]`, `[Tier 2]`, `[Tier 3]`).
5. **No Test Folders in Baseline:** The generated project focuses purely on runnable source code (`src/`). Tests are not scaffolded in the baseline setup.
6. **Strict Enum-First Discipline:** The system replaces all magic strings and static response text with typed enums.
7. **Route-Level Zod Validation:** Zod schemas are defined in module DTOs and validated strictly in route middleware via `validateData(schema)`. Services contain zero `.safeParse()` calls.
8. **Canonical API Response Entity:** All endpoints return `{ status: boolean, statusCode: number, message: string, data: T }` conforming to `APIResponseEntity<T>`.
9. **Confirmation Step:** Compile responses into `project-spec.yaml` and require human confirmation before drafting the generation plan.

---

## Step 0: Target Project Folder Path
- **Question:** "What folder path should the new backend project be generated in?"
- **Default:** `../<project-name>` (an external directory outside the standards repository).
- **Mandatory Validation:** The path MUST NOT point to the root of the standards repository.

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
  - `shared-utils` [Tier 1] *(Default)* — Shared package named `shared-utils` containing enums, constants, route registrars, and error handlers. Structured so it can live as a monorepo workspace package (`packages/shared-utils`) or an independent git repository.
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
