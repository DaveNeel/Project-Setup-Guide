# Database Engine: Microsoft SQL Server (Tier 2 Delta)

## 1. Purpose
Defines the engine-specific configuration delta for Microsoft SQL Server (MSSQL) relative to the PostgreSQL Tier 1 standard.

---

## 2. Applies when
- `database_engine: mssql` is selected in Step 3.

---

## 3. Interaction Notes

| Upstream Choice | Impact on MSSQL |
|:---|:---|
| Base Standard | Inherits from `standards/03-database/relational-common.md` and `standards/03-database/engines/postgres.md`. |
| Step 4 (Data Access) | Uses `mssql` / `tedious` driver with TypeORM, Prisma, or Drizzle. |

---

## 4. Contract (Delta from PostgreSQL)

This standard is identical to `standards/03-database/engines/postgres.md`, **EXCEPT**:

1. **Driver & Port:**
   - Driver: `mssql` (backed by `tedious`).
   - Default Port: `1433` (instead of `5432`).
   - Connection URL: `sqlserver://${DB_HOST}:${DB_PORT};database=${DB_NAME};user=${DB_USER};password=${DB_PASSWORD};encrypt=true;trustServerCertificate=true`.
2. **Schema Namespace:**
   - Default schema namespace is `dbo`. Explicit table references SHOULD specify schema: `dbo.users`.
3. **Primary Key Data Types:**
   - Auto-incrementing keys use `INT IDENTITY(1,1)` or `BIGINT IDENTITY(1,1)`.
   - UUID keys use the `UNIQUEIDENTIFIER` data type with default `NEWID()` or `NEWSEQUENTIALID()`.
4. **Pagination Syntax:**
   - SQL pagination MUST use `OFFSET n ROWS FETCH NEXT m ROWS ONLY` and requires an explicit `ORDER BY` clause.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// MSSQL health check probe
export async function checkMssqlHealth(pool: ConnectionPool): Promise<boolean> {
  const request = pool.request();
  const result = await request.query('SELECT 1 AS status');
  return result.recordset[0]?.status === 1;
}
```

---

## 6. Agent Output Checklist
- [ ] Dependencies include `mssql` or `tedious`.
- [ ] Default port configured to 1433.
- [ ] UUID fields use `UNIQUEIDENTIFIER` with `NEWID()`.

---

## 7. Common Mistakes
- **Missing ORDER BY in Offset Queries:** MSSQL throws syntax error on `OFFSET` without an active `ORDER BY`.
- **SSL Certificate Rejection:** Failing to set `trustServerCertificate: true` in local development environments.
