# Database Engine: MySQL (Tier 2 Delta)

## 1. Purpose
Defines the engine-specific configuration delta for MySQL relative to the PostgreSQL Tier 1 standard.

---

## 2. Applies when
- `database_engine: mysql` is selected in Step 3.

---

## 3. Interaction Notes

| Upstream Choice | Impact on MySQL |
|:---|:---|
| Base Standard | Inherits from `standards/03-database/relational-common.md` and `standards/03-database/engines/postgres.md`. |
| Step 4 (Data Access) | Uses `mysql2` driver with TypeORM, Prisma, or Drizzle. |

---

## 4. Contract (Delta from PostgreSQL)

This standard is identical to `standards/03-database/engines/postgres.md`, **EXCEPT**:

1. **Driver & Port:**
   - Driver: `mysql2` (instead of `pg`).
   - Default Port: `3306` (instead of `5432`).
   - Connection URL: `mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}`.
2. **Boolean Data Type:**
   - MySQL lacks a native `BOOLEAN` type. Booleans MUST be defined as `TINYINT(1)`, with `0` representing false and `1` representing true.
3. **Table Name Case Sensitivity:**
   - Table names on Linux platforms are case-sensitive by default based on file systems. Migration scripts MUST strictly use lowercase `snake_case` table names.
4. **Fractional Seconds in Timestamps:**
   - Datetime fields requiring millisecond accuracy MUST be declared as `DATETIME(6)` or `TIMESTAMP(6)`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// MySQL health check query
export async function checkMySqlHealth(pool: Pool): Promise<boolean> {
  const [rows] = await pool.query('SELECT 1 AS status');
  return (rows as any)[0]?.status === 1;
}
```

---

## 6. Agent Output Checklist
- [ ] Dependencies specify `mysql2`.
- [ ] Default port configured to 3306.
- [ ] Table definitions use lowercase snake_case consistently.

---

## 7. Common Mistakes
- **Driver Mismatch:** Installing legacy `mysql` package instead of `mysql2`.
- **Case Inconsistency:** Writing `SELECT * FROM Users` on Windows which fails when deployed on Linux.
