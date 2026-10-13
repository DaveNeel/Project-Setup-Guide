# Compatibility Matrix & Rejection Engine

This document defines all compatible combinations, disallowed configurations, and the **exact rejection messages** the AI agent MUST output if an invalid configuration is encountered.

---

## 1. Mandatory Verification Rule

> [!IMPORTANT]
> **VERIFY AT GENERATION TIME:**
> The AI agent MUST NOT treat ORM-to-database engine compatibility tables as immutable dogma. At generation time, the agent MUST verify current official documentation for the chosen ORM/driver package (e.g., TypeORM, Prisma, Drizzle, Mongoose). The tables below represent expected support baselines to verify against, not static facts.

---

## 2. Expected Engine & ORM Compatibility Matrix

| Database Engine | TypeORM (T1) | Mongoose (T1) | Prisma (T2) | Drizzle (T2) | Status / Rule |
|:---|:---:|:---:|:---:|:---:|:---|
| **PostgreSQL** | Compatible | Incompatible | Compatible | Compatible | Standard relational SQL stack |
| **MySQL** | Compatible | Incompatible | Compatible | Compatible | Standard relational SQL stack |
| **MSSQL** | Compatible | Incompatible | Compatible | Compatible | Standard relational SQL stack |
| **MongoDB** | Incompatible | Compatible | Incompatible* | Incompatible | Strictly Mongoose for NoSQL Tier 1 |

*\*Note: While Prisma has experimental/limited MongoDB support, in this standard MongoDB data persistence is strictly standardized on Mongoose (Tier 1).*

---

## 3. Incompatible Combinations & Exact Rejection Messages

If a user requests or inputs any of the following combinations, the agent MUST immediately reject the configuration with the **verbatim rejection message** specified below.

### Matrix Violation 1: MongoDB with SQL ORM
- **Trigger:** `database_engine: mongodb` paired with `data_access: typeorm | prisma | drizzle`.
- **Exact Agent Rejection Message:**
  > `"ERROR [COMPATIBILITY_INVALID_ORM]: You selected MongoDB as the database engine, but '${data_access}' is a relational SQL data mapper. For MongoDB, the only supported data-access layer is 'mongoose' [Tier 1]. Please update Step 4 to 'mongoose' or select a relational database in Step 3."`

### Matrix Violation 2: Relational Engine with Mongoose
- **Trigger:** `database_engine: postgres | mysql | mssql` paired with `data_access: mongoose`.
- **Exact Agent Rejection Message:**
  > `"ERROR [COMPATIBILITY_INVALID_ODM]: You selected '${database_engine}' (SQL), but Mongoose is an ODM designed exclusively for MongoDB document databases. Please select a compatible SQL data-access layer: 'typeorm' [Tier 1], 'prisma' [Tier 2], or 'drizzle' [Tier 2]."`

### Matrix Violation 3: Monolith with Microservice Steps
- **Trigger:** `architecture: monolith | modular-monolith` paired with `db_strategy`, `services`, or `inter_service_comm`.
- **Exact Agent Rejection Message:**
  > `"ERROR [COMPATIBILITY_INVALID_TOPOLOGY]: Steps 5 (DB Strategy), 6 (Service Boundaries), and 7 (Inter-Service Comm) apply only to microservices architectures. A monolith operates as a single deployable unit with an in-process modular structure and a unified database connection. Removing these fields and routing to 'monolith-common-folder'."`

### Matrix Violation 4: Monolith with Shared Utils Package
- **Trigger:** `architecture: monolith | modular-monolith` paired with `shared_code_strategy: shared-utils`.
- **Exact Agent Rejection Message:**
  > `"ERROR [COMPATIBILITY_INVALID_SHARED_CODE]: The 'shared-utils' package is designed for multi-service architectures. For a monolithic application, shared code MUST reside in the internal 'src/common/' folder pursuant to 'standards/08-shared-code/monolith-common-folder.md'."`

### Matrix Violation 5: Selection of Tier 3 (Unsupported) Option
- **Trigger:** User chooses `inter_service_comm: grpc`, `config_strategy: vault`, `auth: session | none`, or `shared_code_strategy: none`.
- **Exact Agent Rejection Message:**
  > `"====================================================================\n[NOT YET SUPPORTED]\nThe selected option '${option_name}' is currently classified under Tier 3 (Not Supported).\nThe standards repository does not yet provide verified contracts or automation rules for this choice.\n====================================================================\nPlease select an active Tier 1 or Tier 2 alternative to proceed, or confirm if you wish to pause."`
