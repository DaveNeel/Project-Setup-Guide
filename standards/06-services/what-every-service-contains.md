# Service Standard: Mandatory Service Components (Tier 1)

## 1. Purpose
Defines the mandatory structural baseline, endpoints, and cross-cutting components that every generated microservice MUST contain.

---

## 2. Applies when
- All microservices in `services/*` when `architecture: microservices` is selected.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Framework Parity | Applies equally whether the service is built on NestJS or Express. |
| Quality Standard | Works in tandem with `standards/11-quality/testing.md` for baseline tests. |

---

## 4. Contract

### Mandatory Directory & File Baseline
Every microservice in `services/<service-name>/` MUST contain:
1. **Manifest & TypeScript Config:**
   - `package.json` with scripts: `"build"`, `"lint"`, and `"start:dev"`.
   - `tsconfig.json` extending root compiler options.
2. **Environment Template:**
   - `.env.example` documenting all required variables with placeholder values.
3. **Fail-Fast Typed Config Module:**
   - `src/config/` containing startup schema validation that crashes if required variables are missing, utilizing `NodeEnvEnum` and `LogLevelEnum`.
4. **Structured JSON Logger:**
   - Logger outputting JSON entries including `timestamp`, `level`, `service`, `correlationId`, `message`.
5. **Centralized Error Envelope Handler:**
   - Exception filter (NestJS) or error middleware (Express) returning the cross-cutting error envelope with `CommonMessagesEnumEntity`.
6. **Single Dynamic Health Check Probe:**
   - Endpoint mounted at `GET /health` performing active database connectivity verification, conforming to `standards/cross-cutting/health-checks.md`.
7. **Pure Source Code Baseline:**
   - Pure runnable source code under `src/`. `test/` folders are omitted from the initial setup.

---

## 5. Illustrative Tree
<!-- illustrative only -->
```
services/order-service/
├── package.json
├── tsconfig.json
├── .env.example
└── src/
    ├── server.ts (or main.ts)
    ├── config/
    ├── database/
    ├── health/
    └── modules/
```

---

## 6. Agent Output Checklist
- [ ] Every service directory contains `.env.example`, `package.json`, and `tsconfig.json`.
- [ ] Single health check endpoint `GET /health` exists and verifies active database connectivity.
- [ ] Structured logger and error envelope handlers are wired into bootstrap.
- [ ] Pure `src/` source code baseline without `test/` folders.

---

## 7. Common Mistakes
- **Missing `.env.example` in Individual Services:** Only providing a single `.env.example` at root instead of per service.
- **Skipping Health Probes in Utility Services:** Omitting health checks on smaller internal services.
- **Inconsistent npm Scripts:** Naming scripts `test:unit` in Service A but `test` in Service B.
