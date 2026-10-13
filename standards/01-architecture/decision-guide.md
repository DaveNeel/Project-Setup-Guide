# Architecture Decision Guide

## 1. Purpose
Provides explicit criteria, trade-offs, and downstream structural impacts to guide the selection between `microservices`, `monolith`, and `modular-monolith`.

---

## 2. Applies when
- Evaluated during **Step 1 (Architecture)** of the intake pipeline.

---

## 3. Interaction Notes

| Upstream Choice | Impact on Downstream Steps |
|:---|:---|
| `microservices` | Enables Steps 5 (DB Strategy), 6 (Service Boundaries), 7 (Inter-Service Comm). Step 8 defaults to `shared-utils`. |
| `monolith` | Skips Steps 5, 6, and 7. Step 8 is locked to `monolith-common-folder`. Single database connection instance. |
| `modular-monolith` | Skips Steps 5, 6, and 7. Step 8 is locked to `monolith-common-folder`. Enforces strict module encapsulation. |

---

## 4. Contract

### Topology Evaluation Criteria
- The AI agent MUST recommend `monolith` when:
  - System scope is a single cohesive domain.
  - Development team is small (1-4 engineers).
  - Low operational complexity is required.
- The AI agent MUST recommend `modular-monolith` when:
  - System has multiple distinct business subdomains but deploys as a single runtime.
  - Strict domain separation is needed without distributed system network overhead.
- The AI agent MUST recommend `microservices` when:
  - Independent deployability, scaling, and team ownership are mandatory.
  - Subdomains require heterogeneous database persistence or decoupled release cadences.

### Structural Downstream Impact Table

| Architectural Dimension | Monolith | Modular Monolith | Microservices |
|:---|:---|:---|:---|
| **Deployable Units** | 1 process | 1 process | $N$ independent services |
| **Monorepo Layout** | Root application | Root application | `services/*` + `packages/*` |
| **Database Connections**| Single DB pool | Single DB pool | DB pool per service |
| **Communication** | In-process DI calls | In-process domain API | HTTP REST / RPC over network |
| **Shared Code** | `src/common/` | `src/common/` | `packages/shared-utils/` workspace |

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```
Topology Decision Gate:
├── Single Deployable Unit?
│   ├── Unified domain modules -> monolith
│   └── Strictly bounded domain contracts -> modular-monolith
└── Multiple Autonomous Deployables?
    └── Independent services + shared workspace packages -> microservices
```

---

## 6. Agent Output Checklist
- [ ] Record chosen architecture topology verbatim in `project-spec.yaml`.
- [ ] If monolith or modular-monolith, verify that Steps 5, 6, and 7 are marked skipped.
- [ ] Validate downstream workspace root layout matches selected architecture.

---

## 7. Common Mistakes
- **Scaffolding Microservices for Simple Monoliths:** Introducing multi-service monorepos when the user selected `monolith`.
- **Failing to Enforce Module Boundaries:** Treating `modular-monolith` as a loose monolith where modules reach into arbitrary private classes.
