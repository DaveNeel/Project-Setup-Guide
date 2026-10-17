# Conformance Checklist (Agent Self-Audit)

This checklist contains binary YES/NO items that the AI code-generation agent MUST verify against the scaffolded codebase before reporting completion.

---

## Intake & Setup Initiation
- [ ] **INIT-01:** Did the agent prompt for the target project directory path before starting generation? `[YES / NO]`
- [ ] **INIT-02:** Is the target directory external to the standards repository? `[YES / NO]`
- [ ] **INIT-03:** Did the agent prompt for the `shared-utils` folder path when microservices architecture was selected? `[YES / NO]`

---

## Architecture & Service Structure
- [ ] **ARCH-01:** Did the agent scaffold a parent orchestrator managing microservices without shared dependency hoisting? `[YES / NO]`
- [ ] **ARCH-02:** Are services isolated with zero relative imports between sibling services? `[YES / NO]`
- [ ] **ARCH-03:** Did the agent omit `test/` folders from the baseline project setup? `[YES / NO]`
- [ ] **ARCH-04:** Does each microservice maintain its own dedicated `node_modules/` directory and independent `package-lock.json`? `[YES / NO]`

---

## Shared Utils & Enum-First Discipline
- [ ] **UTIL-01:** Is the common library named `shared-utils` and structured to be extractable to a standalone repository? `[YES / NO]`
- [ ] **UTIL-02:** Are enums defined and used for all HTTP methods (`HTTPRequestMethodsEnumEntity`) and status codes (`HttpStatusCodesEnumEntity`)? `[YES / NO]`
- [ ] **UTIL-03:** Are static response messages replaced with enums (`CommonMessagesEnumEntity`)? `[YES / NO]`
- [ ] **UTIL-04:** Are environments and log levels typed via enums (`NodeEnvEnum`, `LogLevelEnum`)? `[YES / NO]`
- [ ] **UTIL-05:** Does every reusable function, middleware, utility, and handler include a 1-line descriptive comment? `[YES / NO]`
- [ ] **UTIL-06:** Does `shared-utils` contain ONLY clean `.ts` files with zero `.d.ts.map` or `.js.map` map files? `[YES / NO]`
- [ ] **UTIL-07:** In microservices' `package.json`, is `shared-utils` referenced via folder path for initial setup with git repository URL metadata? `[YES / NO]`

---

## Declarative Routing & Controller Handlers
- [ ] **ROUT-01:** Are routes defined as `RouteDefinition[]` and mounted via `registerRoutes(router, routes)`? `[YES / NO]`
- [ ] **ROUT-02:** Are controller actions wrapped with `handleRequestWithResponse`? `[YES / NO]`
- [ ] **ROUT-03:** Are procedural try/catch controller methods and manual `router.get()` calls completely absent? `[YES / NO]`
- [ ] **ROUT-04:** Are controllers strictly limited to HTTP dispatch, parameter extraction, and envelope formatting, with ZERO domain business logic placed in controllers? `[YES / NO]`
- [ ] **ROUT-05:** Do domain services receive pure DTOs/primitives without accepting Express `req` or `res` objects? `[YES / NO]`

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
- [ ] **DATA-04:** If `shared-db` is selected, are database schemas, entities, migrations, and parameterized connection factory `createDataSource(dbConfig)` housed in `shared-utils` with zero secrets in `shared-utils`? `[YES / NO]`
- [ ] **DATA-05:** Are database migrations, schema generation, status checks, runs, and rollbacks managed exclusively inside `shared-utils` (via `migration-runner.ts` and in-memory parameter injection), with zero migration scripts or folders in microservices? `[YES / NO]`

---

## Verification & Provenance
- [ ] **VERI-01:** Does `npm run build` execute with zero TypeScript compiler errors? `[YES / NO]`
- [ ] **VERI-02:** Does `npm run lint` execute with zero ESLint errors? `[YES / NO]`
- [ ] **VERI-03:** Is root `README.md` stamped with `Scaffolded using AI-Ready Project Standards v0.4.0`? `[YES / NO]`
- [ ] **VERI-04:** Was a slim project-level `AGENTS.md` generated inside the target repository? `[YES / NO]`
- [ ] **VERI-05:** Was the scaffolding execution performed autonomously without asking for redundant intermediate permissions once the generation plan was approved? `[YES / NO]`
