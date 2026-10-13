# Conformance Checklist (Agent Self-Audit)

This checklist contains binary YES/NO items that the AI code-generation agent MUST verify against the scaffolded codebase before reporting completion.

---

## Intake & Setup Initiation
- [ ] **INIT-01:** Did the agent prompt for the target project directory path before starting generation? `[YES / NO]`
- [ ] **INIT-02:** Is the target directory external to the standards repository? `[YES / NO]`

---

## Architecture & Service Structure
- [ ] **ARCH-01:** Did the agent scaffold a monorepo workspace with `packages/shared-utils` and `services/*`? `[YES / NO]`
- [ ] **ARCH-02:** Are services isolated with zero relative imports between sibling services? `[YES / NO]`
- [ ] **ARCH-03:** Did the agent omit `test/` folders from the baseline project setup? `[YES / NO]`

---

## Shared Utils & Enum-First Discipline
- [ ] **UTIL-01:** Is the common library named `shared-utils` and structured to be extractable to a standalone repository? `[YES / NO]`
- [ ] **UTIL-02:** Are enums defined and used for all HTTP methods (`HTTPRequestMethodsEnumEntity`) and status codes (`HttpStatusCodesEnumEntity`)? `[YES / NO]`
- [ ] **UTIL-03:** Are static response messages replaced with enums (`CommonMessagesEnumEntity`)? `[YES / NO]`
- [ ] **UTIL-04:** Are environments and log levels typed via enums (`NodeEnvEnum`, `LogLevelEnum`)? `[YES / NO]`
- [ ] **UTIL-05:** Does every reusable function, middleware, utility, and handler include a 1-line descriptive comment? `[YES / NO]`

---

## Declarative Routing & Controller Handlers
- [ ] **ROUT-01:** Are routes defined as `RouteDefinition[]` and mounted via `registerRoutes(router, routes)`? `[YES / NO]`
- [ ] **ROUT-02:** Are controller actions wrapped with `handleRequestWithResponse`? `[YES / NO]`
- [ ] **ROUT-03:** Are procedural try/catch controller methods and manual `router.get()` calls completely absent? `[YES / NO]`

---

## Validation & API Response Envelopes
- [ ] **VAL-01:** Are Zod schemas validated exclusively in route middleware via `validateData(schema)`? `[YES / NO]`
- [ ] **VAL-02:** Are service methods completely free of `.safeParse()` or `.parse()` calls? `[YES / NO]`
- [ ] **RESP-01:** Do all responses strictly conform to `APIResponseEntity` (`status`, `statusCode`, `message`, `data`)? `[YES / NO]`

---

## Single Dynamic Health Check Probe
- [ ] **HLTH-01:** Does each service expose exactly ONE health check endpoint (`GET /health`)? `[YES / NO]`
- [ ] **HLTH-02:** Does the health endpoint actively probe database connectivity (not just return static JSON)? `[YES / NO]`
- [ ] **HLTH-03:** Does it return HTTP 503 if the database is disconnected? `[YES / NO]`

---

## Database & Data Access Boundary
- [ ] **DATA-01:** Does each aggregate sit behind an explicit repository interface (`I*Repository`)? `[YES / NO]`
- [ ] **DATA-02:** Are controllers and routers completely free of direct database models, queries, or ORM entities? `[YES / NO]`
- [ ] **DATA-03:** If Mongoose is used, are schemas configured with `strict: true`, `timestamps: true`, and `versionKey: false`? `[YES / NO]`

---

## Verification & Provenance
- [ ] **VERI-01:** Does `npm run build` execute with zero TypeScript compiler errors? `[YES / NO]`
- [ ] **VERI-02:** Does `npm run lint` execute with zero ESLint errors? `[YES / NO]`
- [ ] **VERI-03:** Is root `README.md` stamped with `Scaffolded using AI-Ready Project Standards v0.3.0`? `[YES / NO]`
- [ ] **VERI-04:** Was a slim project-level `AGENTS.md` generated inside the target repository? `[YES / NO]`
