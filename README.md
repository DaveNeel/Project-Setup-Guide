# AI-Ready Project Standards

A contract-driven, rules-only architectural framework designed for autonomous AI agents (Claude Code, Cursor, Gemini CLI, Codex) to generate production-ready backend project setups step-by-step.

> **CRITICAL REPOSITORY RULE:** This repository contains **strictly standards, contracts, decision trees, intake schemas, and checklists**. It contains **ZERO** application code, runnable scripts (`.sh`, `.ts`, `.js`), or Dockerfiles. All code examples in standards are non-runnable, illustrative contract shapes strictly $\le 15$ lines.

---

## Purpose

When building backend projects, AI agents often produce inconsistent directory layouts, ad-hoc error shapes, mixed data-access patterns, or loose configuration handling.

This repository serves as the **authoritative rulebook**. An AI agent reads these specifications along with a user's step-wise intake decisions, drafts a strict generation plan, and scaffolds the target project in an external directory following every contract to the letter.

---

## The Decision Pipeline

Target project setups are decided along a strictly ordered, dependency-aware pipeline:

| Step | Decision | Tier 1 (Full Contract) | Tier 2 (Delta from T1) | Tier 3 (NOT SUPPORTED) | Notes / Dependencies |
|:---|:---|:---|:---|:---|:---|
| **0** | **Target Folder** | Explicit path (e.g. `../my-app`) | — | — | External directory prompt |
| **1** | **Architecture** | `microservices`, `monolith` | `modular-monolith` | — | Base topology |
| **2** | **Runtime & Framework** | `express`, `nestjs` | — | — | Declarative router / controllers |
| **3** | **Database Engine** | `mongodb`, `postgres` | `mysql`, `mssql` | — | Relational vs NoSQL |
| **4** | **Data-Access Layer** | `mongoose` (NoSQL), `typeorm` (SQL) | `prisma`, `drizzle` | Other ORMs | Verifies docs at gen-time |
| **5** | **DB Strategy** | `db-per-service`, `shared-db` | — | — | Microservices only |
| **6** | **Service Boundaries** | `service-boundaries`, `what-every-service-contains` | — | — | Microservices only |
| **7** | **Inter-service Comm** | `rest` | `rabbitmq` | `grpc` | Microservices only |
| **8** | **Shared Code Strategy** | `shared-utils`, `monolith-common-folder` | `git-submodule`, `published-private-packages` | `none` | Standalone repo readiness |
| **9** | **Config & Secrets** | `typed-config` + `env-naming` | `aws-secrets-manager` | `vault` | Fail-fast boot validation |
| **10**| **Authentication** | `jwt` | — | `session`, `none` | Bearer token contracts |
| **11**| **Quality Baseline** | `lint-format`, `code-style`, `testing` | — | — | ESLint, strict TS (tests deferred in setup) |
| **X** | **Cross-Cutting** | `error-handling`, `logging`, `api-conventions`, `validation`, `health-checks` | — | — | Enums everywhere, single /health probe |

---

## Support Tiers

- **Tier 1 (Full):** Complete contract-level specification, mandatory folder trees, naming rules, short illustrative snippets ($\le 15$ lines), and checklist items.
- **Tier 2 (Delta):** Differential document stating `"Identical to the Tier 1 equivalent, EXCEPT:"` listing only explicit deltas.
- **Tier 3 (Not Supported):** Displays a `"NOT YET SUPPORTED"` banner. If selected by the user, the agent MUST stop and prompt for clarification rather than hallucinate.

---

## How AI Agents Interact With This Repository

1. **Entry Point:** The agent begins by reading [AGENTS.md](AGENTS.md).
2. **Intake:** The agent runs the step-by-step interview defined in [intake/step-wise-intake.md](intake/step-wise-intake.md) or ingests a populated [project-spec.yaml](intake/project-spec.template.yaml).
3. **Validation:** Ingested choices are validated against [intake/decision-tree.md](intake/decision-tree.md) and [intake/compatibility-matrix.md](intake/compatibility-matrix.md).
4. **Selective Loading:** Using the Routing Table in `AGENTS.md`, the agent loads **only** the standards corresponding to chosen options.
5. **Planning & Approval:** The agent outputs a structured generation plan (directory tree, dependency versions, ports, env keys) and **waits for human approval**.
6. **External Generation:** The agent scaffolds the project into a separate directory, installs dependencies, verifies build/lint/test, audits against [checklists/conformance-checklist.md](checklists/conformance-checklist.md), and stamps the output with `VERSION`.

---

## Repository Layout

```
project-setup/
├── AGENTS.md                  # Universal entry point for all AI agents: routing table & engagement rules
├── README.md                  # This file
├── CHANGELOG.md               # Version history
├── VERSION                    # Standards release version
├── intake/                    # Interactive questionnaire, schema, template, matrices
├── standards/                 # Step-wise standards & cross-cutting contracts
├── folder-structure/          # Literal ASCII folder trees
├── workflows/                 # Numbered generation & maintenance procedures
├── checklists/                # Conformance (agent audit) & human review checklists
├── decisions/                 # Architectural Decision Records (ADRs)
└── .claude/                   # Claude Code skill and slash command
```
