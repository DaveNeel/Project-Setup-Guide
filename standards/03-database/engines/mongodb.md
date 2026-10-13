# Database Engine: MongoDB (Tier 1)

## 1. Purpose
Defines specific engine configurations, drivers, health probes, and known runtime behaviors for MongoDB.

---

## 2. Applies when
- `database_engine: mongodb` is selected in Step 3.

---

## 3. Interaction Notes

| Upstream Choice | Impact on MongoDB Configuration |
|:---|:---|
| Step 4: `mongoose` | Uses Mongoose ODM connection layer (`mongoose.connect`). |
| Step 5: `db-per-service` | Each service connects to a unique database path on the cluster. |

---

## 4. Contract

### Engine Specifications
- **Driver / ODM:** `mongoose` (with `@types/mongoose` if separate).
- **Default Port:** `27017`.
- **Environment Key:** `MONGODB_URI` (or structured host/port/db).
- **Connection URL Format:**
  `mongodb://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}?authSource=admin`
- **Health Check Probe:**
  - Command: `mongoose.connection.db.admin().ping()`
  - Timeout: 3,000ms.
  - Success Condition: Returns `{ ok: 1 }`.

### Engine Quirks & Conventions
- **Transaction Requirements:** MongoDB multi-document ACID transactions require a replica set. Standalone single nodes in local dev do not support multi-document transactions unless replica set initialization is executed.
- **Connection Ready States:** The application MUST verify `connection.readyState === 1` before accepting incoming HTTP traffic.
- **ObjectId Serialization:** Never return raw BSON `ObjectId` objects in HTTP payloads; convert to standard string `id`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// MongoDB health probe contract
export async function checkMongoHealth(connection: Connection): Promise<boolean> {
  if (connection.readyState !== 1) return false;
  const res = await connection.db.admin().ping();
  return res.ok === 1;
}
```

---

## 6. Agent Output Checklist
- [ ] Dependencies include `mongoose`.
- [ ] Health check implementation runs `admin().ping()`.
- [ ] `MONGODB_URI` validated in configuration schema.

---

## 7. Common Mistakes
- **Assuming Standalone Transactions Work:** Calling `session.startTransaction()` on a non-replica-set local MongoDB instance.
- **Failing to Handle Reconnections:** Crashing the Node.js process immediately when an initial connection drops.
