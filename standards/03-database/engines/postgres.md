# Database Engine: PostgreSQL (Tier 1)

## 1. Purpose
Defines specific engine configurations, drivers, health queries, and known runtime behaviors for PostgreSQL.

---

## 2. Applies when
- `database_engine: postgres` is selected in Step 3.

---

## 3. Interaction Notes

| Upstream Choice | Impact on PostgreSQL Configuration |
|:---|:---|
| Step 4: `typeorm` | Requires `pg` driver; uses TypeORM Postgres driver adapter. |
| Step 4: `prisma` | Uses Prisma PostgreSQL provider in schema. |
| Step 4: `drizzle` | Uses `drizzle-orm/node-postgres` with `pg` pool. |

---

## 4. Contract

### Engine Specifications
- **Driver:** `pg` (with `@types/pg`).
- **Default Port:** `5432`.
- **Environment Key:** `DATABASE_URL` or structured `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`.
- **Connection URL Format:**
  `postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}?sslmode=prefer`
- **Health Check Probe:**
  - SQL Query: `SELECT 1;`
  - Timeout: 3,000ms.

### Engine Quirks & Conventions
- **UUID Generation:** For UUID primary keys, migrations MUST use native `gen_random_uuid()` (PostgreSQL 13+) or enable `CREATE EXTENSION IF NOT EXISTS "uuid-ossp";`.
- **Timestamps:** Timestamps MUST be stored as `TIMESTAMP WITH TIME ZONE` (`TIMESTAMPTZ`) to preserve UTC precision.
- **Connection Pools:** Max connections in local dev defaults to 10; serverless/microservices configurations MUST configure connection pooling.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// PostgreSQL health probe contract
export async function checkPostgresHealth(pool: Pool): Promise<boolean> {
  const client = await pool.connect();
  try {
    const res = await client.query('SELECT 1 AS status');
    return res.rows[0]?.status === 1;
  } finally {
    client.release();
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Dependencies include `pg` and `@types/pg`.
- [ ] Health check implementation executes `SELECT 1`.
- [ ] Timestamp columns use `timestamp with time zone`.

---

## 7. Common Mistakes
- **Using naive `TIMESTAMP`:** Using `without time zone`, stripping UTC offset data.
- **Forgetting UUID extension on older Postgres:** Attempting `uuid_generate_v4()` without loading the extension.
