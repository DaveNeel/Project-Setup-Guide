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

### Table & Collection Ownership
- Every table or collection MUST have exactly ONE owning service.
- Relational databases MUST enforce boundaries using separate SQL schemas (e.g. `auth.users`, `billing.invoices`) or strict table prefixes (e.g. `auth_users`, `billing_invoices`).
- Cross-service database joins (e.g. `SELECT * FROM auth.users JOIN billing.invoices`) are STRICTLY PROHIBITED.
- Service A MUST NOT write to or read from tables owned by Service B. Data retrieval MUST occur via Service B's public API.

### Migration Ownership
- Each service MUST manage migrations only for the tables it explicitly owns.
- Migration tables (e.g., `typeorm_metadata`, `flyway_schema_history`) MUST be segregated per service (e.g., `migrations_auth`, `migrations_billing`) to prevent migration lock contention.
- Dropping or modifying columns used by other services' read replicas requires backward-compatible multi-phase schema rollouts.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Shared-DB schema ownership boundary
@Entity({ name: 'users', schema: 'auth' })
export class UserEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  // Billing entities live under schema: 'billing' and CANNOT be joined here
}
```

---

## 6. Agent Output Checklist
- [ ] Explicit schema namespacing (or prefixing) defined per service.
- [ ] Separate migration tracking tables configured in each service's ORM datasource.
- [ ] Zero cross-schema foreign keys or joins across domain boundaries.

---

## 7. Common Mistakes
- **Cross-Service SQL Joins:** Writing queries that join tables owned by different services.
- **Colliding Migration Locks:** Using the default `migrations` table across multiple services in the same database.
- **Shared Mutable Tables:** Allowing two services to write records to the same table.
