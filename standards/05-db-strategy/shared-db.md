# Database Strategy Standard: Shared-Database (Tier 1)

## 1. Purpose
Defines boundaries, table ownership, and migration discipline when multiple microservices share a single physical database cluster.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1, AND
- `db_strategy: shared-db` is selected in Step 5.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Upstream Constraint | Microservices sharing a database MUST NOT treat it as an integration point. |
| Step 3 (Database Engine) | Relational engines use named schemas or table prefixes; MongoDB uses isolated collection prefixes. |

---

## 4. Contract

### Centralized Schema, Migrations & Connection in `shared-utils`
- When `db_strategy: shared-db` is used in a microservices architecture:
  - All database schemas and ORM entities across the microservices MUST reside centrally inside `shared-utils` (e.g. `src/database/entities/`).
  - **Zero Migrations in Microservices:** Microservices are STRICTLY PROHIBITED from defining migrations, executing migrations at boot, or containing migration commands in their local `package.json`.
  - **Exclusively Managed in `shared-utils`:** Migration generation, status inspection, execution, and rollbacks MUST reside and execute exclusively within `shared-utils`.
  - **Parameterized Connection Factory:** `shared-utils` MUST export a connection factory function (e.g. `createDataSource(dbConfig: DBConfig): DataSource`) that accepts typed connection configuration as an argument.
  - **Zero Secrets on Disk in `shared-utils`:** `shared-utils` MUST NOT contain `.env` files or hardcoded connection credentials.
  - **Microservice Secret Ownership:** Each microservice defines and validates its own database connection credentials (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, etc.) in its local `.env` and `src/config/`, passing them as parameters to `createDataSource(dbConfig)` at boot.

### Table & Collection Ownership
- Every table or collection MUST have exactly ONE owning service.
- Relational databases MUST enforce boundaries using separate SQL schemas (e.g. `auth.users`, `billing.invoices`) or strict table prefixes (e.g. `auth_users`, `billing_invoices`).
- Cross-service database joins (e.g. `SELECT * FROM auth.users JOIN billing.invoices`) are STRICTLY PROHIBITED.
- Service A MUST NOT write to or read from tables owned by Service B. Data retrieval MUST occur via Service B's public API.

### Centralized Migration Lifecycle (In-Memory Runner)
- **Lifecycle Commands in `shared-utils`:**
  - `npm run migrate:status`: Verifies executed vs pending migrations.
  - `npm run migrate:generate -- <MigrationName>`: Diffs centralized entities against target DB and generates a new migration.
  - `npm run migrate:run`: Applies pending migrations sequentially within transactions.
  - `npm run migrate:revert`: Safely rolls back the most recent migration.
- **In-Memory Parameter Injection (Zero Temporary Secret Files):**
  - Connection parameters (`host`, `port`, `username`, `password`, `database`) are supplied at runtime via CLI arguments (`npm run migrate:<action> -- ...`) or standard environment variables (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`).
  - The migration runner (`src/database/migration-runner.ts`) parses the inputs and injects them into in-memory process environment variables.
  - `src/database/migration-data-source.ts` reads the configuration in-memory. Writing temporary credential files to disk (`temp-data-source.ts`) is PROHIBITED to prevent credential leakage in logs or filesystem artifacts.
- Dropping or modifying columns used by other services' read operations requires backward-compatible multi-phase schema rollouts.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// src/database/migration-data-source.ts in shared-utils (in-memory config)
import { DataSource } from 'typeorm';
import { entities } from './entities';
import * as path from 'path';

export const MigrationDataSource = new DataSource({
  type: 'mssql',
  host: process.env.DB_HOST || 'localhost',
  port: Number(process.env.DB_PORT) || 1433,
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  synchronize: false,
  logging: true,
  entities: entities,
  migrations: [path.join(__dirname, 'migrations/*{.ts,.js}')],
  options: { encrypt: true, trustServerCertificate: true }
});
```

---

## 6. Agent Output Checklist
- [ ] Database entities and migrations reside 100% centrally in `shared-utils`.
- [ ] Microservices contain zero migration directories or migration CLI scripts.
- [ ] `shared-utils` exposes `migrate:status`, `migrate:run`, `migrate:generate`, and `migrate:revert`.
- [ ] Migration runner supports in-memory parameter injection without writing temporary secret files to disk.
- [ ] Explicit schema namespacing (or prefixing) defined per service.
- [ ] Zero cross-schema foreign keys or joins across domain boundaries.

---

## 7. Common Mistakes
- **Managing Migrations at Microservice Level:** Scaffolding migration folders, scripts, or execution in individual microservices when a shared database is used, leading to schema drift, conflicting migration locks, and lack of a single source of truth.
- **Temporary Secret Files on Disk:** Writing database credentials to a temporary file (`temp-data-source.ts`) that can linger if the process crashes or is killed.
- **Cross-Service SQL Joins:** Writing queries that join tables owned by different services.
- **Shared Mutable Tables:** Allowing two services to write records to the same table.
