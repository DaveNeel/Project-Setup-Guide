# Database Standard: Relational Common (SQL)

## 1. Purpose
Defines shared database contracts for relational database engines, covering connection management, migration standards, naming rules, and transaction boundaries.

---

## 2. Applies when
- `database_engine` is any of `postgres`, `mysql`, or `mssql` in Step 3.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Step 4 (Data Access) | Implemented by TypeORM, Prisma, or Drizzle according to Step 4 choice. |
| Step 5 (DB Strategy) | Dictates whether migrations and schemas are per-service or unified across services. |

---

## 4. Contract

### Connection Pooling & Lifecycle
- Applications MUST use managed connection pooling with explicit min and max connection thresholds.
- Default pool bounds MUST be configurable via environment variables (`DB_POOL_MIN`, `DB_POOL_MAX`).
- Connection acquisition timeout MUST be configured (default: 10,000ms).
- Graceful shutdown hooks MUST close all open connection pools on `SIGINT` / `SIGTERM`.

### Naming Conventions
- Database table names MUST be plural and `snake_case` (e.g. `users`, `order_items`).
- Column names MUST be `snake_case` (e.g. `first_name`, `created_at`).
- Primary keys MUST be named `id` (UUIDv4 or auto-incrementing integer).
- Foreign key columns MUST follow the pattern `<singular_table>_id` (e.g. `user_id`, `order_id`).
- Timestamps MUST include `created_at` and `updated_at` stored in UTC.

### Migrations & Transactions
- Database schema mutations MUST be executed via explicit migration files committed to git.
- Running automatic migrations on production boot (`synchronize: true`) is STRICTLY PROHIBITED.
- Cross-entity state mutations requiring atomicity MUST execute inside an explicit database transaction managed at the service or unit-of-work layer.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Migration contract illustration
export class CreateUsersTable1710000000000 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(new Table({
      name: 'users',
      columns: [
        { name: 'id', type: 'uuid', isPrimary: true, isGenerated: true },
        { name: 'email', type: 'varchar', isUnique: true, isNullable: false },
        { name: 'created_at', type: 'timestamp with time zone', default: 'NOW()' }
      ]
    }));
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Schema naming adheres strictly to snake_case and plural tables.
- [ ] `synchronize: false` is configured for production environments.
- [ ] Explicit migration script (`npm run migration:run`) defined in `package.json`.

---

## 7. Common Mistakes
- **Enabling Auto-Sync in Production:** Setting `synchronize: true` in production database configuration.
- **CamelCase Column Names in SQL:** Creating SQL columns like `firstName` instead of `first_name`.
- **Missing UTC Timestamps:** Storing timestamps with local server timezone offsets.
