# Decision Tree

This document formalizes the decision graph. The AI agent MUST consult this graph to determine which options are valid at each step based on preceding choices.

---

## 1. Top-Level Decision Flowchart

```mermaid
graph TD
    Start([User Initiates Intake]) --> Step0[Step 0: Target Folder Path & Shared Utils Path]
    Step0 --> Step1{Step 1: Architecture}
    
    Step1 -->|microservices| MS[Microservices Pipeline]
    Step1 -->|monolith / modular-monolith| Mono[Monolith Pipeline]
    
    MS --> Step2[Step 2: Framework<br>express | nestjs]
    Mono --> Step2
    
    Step2 --> Step3{Step 3: Database Engine}
    
    Step3 -->|postgres / mysql / mssql| SQLBranch[SQL Engine Branch]
    Step3 -->|mongodb| NoSQLBranch[NoSQL Engine Branch]
    
    SQLBranch --> Step4_SQL[Step 4: Data Access<br>typeorm | prisma | drizzle]
    NoSQLBranch --> Step4_NoSQL[Step 4: Data Access<br>mongoose ONLY]
    
    Step4_SQL --> GateMS1{Is Microservices?}
    Step4_NoSQL --> GateMS1
    
    GateMS1 -->|Yes| Step5[Step 5: DB Strategy<br>db-per-service | shared-db<br>shared-db: entities & connection in shared-utils]
    GateMS1 -->|No: Monolith| SkipMS[Skip Steps 5, 6, 7]
    
    Step5 --> Step6[Step 6: Service List & Boundaries]
    Step6 --> Step7[Step 7: Inter-Service Comm<br>rest | rabbitmq | grpc:T3]
    
    Step7 --> Step8_MS[Step 8: Shared Code<br>shared-utils: folder path + git repo URL<br>pure .ts only]
    SkipMS --> Step8_Mono[Step 8: Shared Code<br>monolith-common-folder]
    
    Step8_MS --> Step9[Step 9: Config & Env<br>typed-config | aws-secrets | vault:T3]
    Step8_Mono --> Step9
    
    Step9 --> Step10[Step 10: Auth<br>jwt | session:T3 | none:T3]
    Step10 --> Confirm([Compile & Confirm project-spec.yaml])
```

---

## 2. Detailed Branching Logic

### Branch 1: Architecture Topology (Step 1)
- If `microservices`:
  - Active steps: All steps (0 through 10).
  - Step 8 default: `shared-utils`.
- If `monolith` or `modular-monolith`:
  - Active steps: 1, 2, 3, 4, 8, 9, 10, 11.
  - Automatically skipped steps:
    - **Step 5 (DB Strategy):** Replaced by single monolith database instance.
    - **Step 6 (Service Boundaries):** Replaced by modular domain folders (`src/modules/<domain>`).
    - **Step 7 (Inter-Service Comm):** In-process function/service injection replaces network IPC.
  - Step 8 resolution: Automatically locked to `monolith-common-folder` (`src/common/`).

### Branch 2: Database Engine & Data Access (Steps 3 & 4)
- If `database_engine` in `['postgres', 'mysql', 'mssql']`:
  - Valid `data_access` options:
    - `typeorm` [Tier 1] *(Default)*
    - `prisma` [Tier 2]
    - `drizzle` [Tier 2]
  - Disallowed options: `mongoose` (rejection reason: MongoDB ODM cannot query relational databases).
- If `database_engine == 'mongodb'`:
  - Valid `data_access` options:
    - `mongoose` [Tier 1] *(Default & Only valid option)*
  - Disallowed options: `typeorm`, `prisma`, `drizzle` (rejection reason: relational ORM patterns invalid for MongoDB document store).

### Branch 3: Shared Code Strategy (Step 8)
- If `architecture == 'microservices'`:
  - Allowed: `shared-utils` [T1] *(Default)*, `git-submodule` [T2], `published-private-packages` [T2], `none` [T3].
- If `architecture` in `['monolith', 'modular-monolith']`:
  - Allowed: `monolith-common-folder` [T1].
  - Disallowed: `shared-utils` (monorepo packages add unnecessary overhead to a single deployable).

### Branch 4: Support Tier Gates
- If any option is Tier 3 (`grpc`, `vault`, `session`, `none` for auth/shared-code):
  - The agent MUST NOT proceed silently.
  - The agent MUST halt intake, output the `"NOT YET SUPPORTED"` banner, and ask the user to choose a Tier 1 or Tier 2 alternative or wait for future support.
