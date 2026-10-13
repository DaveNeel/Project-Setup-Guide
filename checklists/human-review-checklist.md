# Human Review Checklist

This checklist contains architectural and operational review items requiring human judgment that cannot be fully evaluated through automated script execution alone.

---

## 1. Domain Boundaries & System Topology
- [ ] **Domain Decomposition:** Are service or module boundaries aligned with business capabilities, avoiding anemic microservices or God-modules?
- [ ] **Transaction Scope:** Do transactions remain within single service boundaries, avoiding distributed locks or two-phase commits?
- [ ] **Data Ownership:** Does each database table or collection have a single, unambiguous owner?

---

## 2. Security & Credentials
- [ ] **Secret Hygiene:** Are all live credentials, API keys, and production passwords stored in a secure secret store, completely absent from source code?
- [ ] **JWT Key Strength:** Is the production `JWT_SECRET` generated with sufficient cryptographic entropy ($\ge 256$ bits)?
- [ ] **Role Permissions:** Are role-based access control (RBAC) boundaries appropriately restrictive for administrative endpoints?

---

## 3. Data Integrity & Migration Safety
- [ ] **Migration Rollback:** Are down migrations tested to ensure backward compatibility and clean rollbacks?
- [ ] **Destructive Mutations:** Do new migrations avoid dropping columns or altering constraints in ways that break active production consumers?
- [ ] **Connection Pooling Limits:** Are database pool sizes tuned for the target infrastructure to prevent exhausting database connection limits?

---

## 4. Resilience & Operational Readability
- [ ] **Downstream Circuit Breaking:** Are timeouts and retries for external services configured to prevent cascading failures?
- [ ] **Log Field Redaction:** Are all PII, sensitive query parameters, and auth tokens correctly redacted from application logs?
- [ ] **Health Probe Scope:** Does the single `GET /health` probe actively verify database connectivity without leaking internal credentials or overwhelming the database?
