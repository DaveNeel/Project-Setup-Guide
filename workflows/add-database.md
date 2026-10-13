# Workflow: Add or Migrate Database

Numbered procedure for adding a database engine, configuring connection lifecycles, and establishing migration pipelines.

---

## 1. Documentation Verification
1. Verify the selected ORM/driver compatibility against current official package documentation.
2. Check that the chosen database engine aligns with the project's data-access layer (SQL $\rightarrow$ TypeORM/Prisma/Drizzle; NoSQL $\rightarrow$ Mongoose).

---

## 2. Dependency Installation
1. Install driver package (e.g. `pg`, `mysql2`, `mssql`, `mongoose`).
2. Install TypeScript type definitions (e.g. `@types/pg`).

---

## 3. Configuration & Environment Schema
1. Add database connection variables to `.env.example` (`DATABASE_URL`, `DB_POOL_MIN`, `DB_POOL_MAX`).
2. Update `src/config/` validation schema to enforce connection URI structure and credential presence.

---

## 4. Connection Lifecycle & Shutdown Hooks
1. Implement database module or connection pool factory.
2. Register graceful shutdown hooks (`onApplicationShutdown` or process `SIGTERM`) to cleanly terminate connection pools.

---

## 5. Migration Pipeline Setup
1. Create dedicated migration directory: `src/database/migrations/`.
2. Configure standalone migration CLI configuration (`data-source.ts` for TypeORM, `drizzle.config.ts` for Drizzle, or `prisma/schema.prisma` for Prisma).
3. Add migration scripts to `package.json`:
   - `"migration:generate"`
   - `"migration:run"`
   - `"migration:revert"`
4. Enforce `synchronize: false` in production configuration.

---

## 6. Health Probe Integration
1. Update single `GET /health` probe in `health.controller.ts` to execute active database connectivity query (`SELECT 1` for SQL, `readyState === 1` or `admin().ping()` for MongoDB).
2. Ensure unhealthy database connection triggers HTTP 503 response.

---

## 7. Repository Layer Integration
1. Define entities/schemas in `src/modules/<domain>/entities/` (or `schemas/`).
2. Implement concrete repository wrapping the ORM client behind the domain repository interface.
3. Verify domain services consume only the interface.

---

## 8. Verification
1. Run `npm run migration:run` to apply migrations (if relational).
2. Run `npm run build` to verify clean compilation with zero type errors.
