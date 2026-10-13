# Database Strategy Standard: Database-Per-Service (Tier 1)

## 1. Purpose
Defines the architectural and isolation contracts for microservices where each service maintains exclusive ownership over its persistent data store.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1, AND
- `db_strategy: db-per-service` is selected in Step 5.

---

## 3. Interaction Notes

| Preceding Decision | Impact on DB-per-Service Strategy |
|:---|:---|
| Step 3 (Database Engine) | Applies to either relational (PostgreSQL, MySQL, MSSQL) or document (MongoDB) databases. |
| Step 6 (Service List) | Services with `own_db: true` instantiate independent database configurations and migrations. |

---

## 4. Contract

### Data Isolation & Access Prohibition
- Each service with `own_db: true` MUST connect to its own dedicated database instance, catalog, or isolated schema.
- Cross-service database access is STRICTLY PROHIBITED. Service A MUST NEVER execute queries, joins, or connection pools against Service B's database.
- Shared database credentials across distinct services are STRICTLY PROHIBITED. Each service MUST define independent connection environment variables (e.g. `SERVICE_A_DB_NAME`, `SERVICE_B_DB_NAME`).

### Migrations & Schema Evolution
- Each service MUST maintain its own independent migration folder within its directory tree: `services/<service-name>/src/database/migrations/`.
- Migrations MUST be executable independently without requiring locks or coordinating runs across other services.
- Database schemas MUST be versioned and deployed strictly in lockstep with the owning service.

### Distributed Data Consistency
- When business transactions span multiple services, services MUST maintain eventual consistency via asynchronous events or choreographed sagas. Two-phase commits (2PC) are PROHIBITED.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```yaml
# Environment segregation contract
# services/auth-service/.env.example
DATABASE_URL=postgresql://auth_user:auth_secret@localhost:5432/auth_service_db

# services/order-service/.env.example
DATABASE_URL=postgresql://order_user:order_secret@localhost:5432/order_service_db
```

---

## 6. Agent Output Checklist
- [ ] Each microservice contains dedicated migration directories.
- [ ] Environment configurations declare distinct database names per service.
- [ ] Zero foreign key constraints or SQL joins referencing tables owned by other services.

---

## 7. Common Mistakes
- **Cross-Service Database Queries:** Querying another service's database directly to avoid creating an API endpoint.
- **Unified Migration Scripts:** Placing all migrations in a central root folder executed as a single batch for all services.
- **Shared DB Credentials:** Using a single superuser credentials string across all microservices.
