# Cross-Cutting Standard: Single Dynamic Health Check Probe

## 1. Purpose
Defines the single unified health check endpoint that dynamically verifies database connectivity and process health.

---

## 2. Applies when
- Every deployable application and microservice.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Endpoint Consolidation | Exactly ONE endpoint (`GET /health`). Dual live/ready endpoints are eliminated. |
| Database Integration | Actively queries database connectivity state (Mongoose `readyState === 1` or SQL `SELECT 1`). |
| Enums | Status values MUST use `HealthStatusEnum` from `shared-utils`. |

---

## 4. Contract

### Endpoint Specification
- Path: `GET /health`.
- Public access: Bypasses authentication guards.
- Dynamic Probe:
  - Executes active database connectivity check.
  - If database is connected: returns HTTP 200 with `{ status: HealthStatusEnum.OK, database: HealthStatusEnum.UP }`.
  - If database is disconnected: returns HTTP 503 with `{ status: HealthStatusEnum.ERROR, database: HealthStatusEnum.DOWN }`.

### Standard Health Response Payload
```json
{
  "status": "ok",
  "timestamp": "2026-10-10T00:00:00.000Z",
  "database": "up"
}
```

### One-Line Function Documentation
- The health probe controller/handler MUST include a 1-line comment explaining its connectivity verification.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Performs active database connectivity check and returns dynamic health status
export const checkHealth = async (_req: Request, res: Response) => {
  const isDbConnected = await checkDatabaseConnection();
  const statusCode = isDbConnected 
    ? HttpStatusCodesEnumEntity.OK 
    : HttpStatusCodesEnumEntity.SERVICE_UNAVAILABLE;

  return res.status(statusCode).json({
    status: isDbConnected ? HealthStatusEnum.OK : HealthStatusEnum.ERROR,
    timestamp: new Date().toISOString(),
    database: isDbConnected ? HealthStatusEnum.UP : HealthStatusEnum.DOWN
  });
};
```

---

## 6. Agent Output Checklist
- [ ] Exactly one health endpoint mounted at `GET /health`.
- [ ] Actively checks database status (does NOT return a static dummy response).
- [ ] Returns HTTP 503 if database check fails.
- [ ] Uses `HealthStatusEnum` and `HttpStatusCodesEnumEntity`.

---

## 7. Common Mistakes
- **Scaffolding Dual Endpoints:** Creating separate `/health/live` and `/health/ready` endpoints.
- **Static Health Responses:** Returning `{ status: 'ok' }` without checking database connectivity.
- **Using String Literals:** Hardcoding status strings like `'ok'` instead of `HealthStatusEnum.OK`.
