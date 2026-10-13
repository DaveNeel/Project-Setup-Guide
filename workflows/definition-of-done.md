# Definition of Done: Generated Project Setup

This document establishes the mandatory acceptance gates that every generated backend project MUST satisfy before an AI agent may mark the task complete.

---

## 1. Zero Build & Compilation Errors
- [ ] `npm run build` (or workspace build command) executes cleanly and exits with code 0.
- [ ] Zero TypeScript compiler errors under `strict: true`.
- [ ] All output artifacts compile to configured `dist/` or `build/` directories.

---

## 2. Zero Static Analysis Violations
- [ ] `npm run lint` executes cleanly and exits with code 0.
- [ ] Zero ESLint errors or unhandled warnings.
- [ ] Code formatting adheres to `.prettierrc`.

---

## 3. Baseline Verification (Tests Deferred for Setup)
- [ ] Baseline setup omits `test/` folders to focus on clean, runnable source code.
- [ ] Verification is gated on `npm run build` (TypeScript compilation) and `npm run lint` (ESLint).
- [ ] Single `GET /health` endpoint verifies active database connectivity dynamically.

---

## 4. Contract Conformance & Coding Standards
- [ ] Universal enums used everywhere (`HTTPRequestMethodsEnumEntity`, `HttpStatusCodesEnumEntity`, `CommonMessagesEnumEntity`, `NodeEnvEnum`, `LogLevelEnum`, `HealthStatusEnum`) — zero static string literals.
- [ ] Express routes declared as `RouteDefinition[]` and mounted via `registerRoutes(router, routes)`.
- [ ] Express controllers wrap service actions via `handleRequestWithResponse((req, res) => service.method(req, res))`.
- [ ] Concise 1-line descriptive comment present on every reusable function, middleware, utility, and handler.
- [ ] Zod schemas validated via `validateData(schema)` route middleware — service methods contain zero `.safeParse()` or `.parse()` calls.
- [ ] All responses (success and error) strictly conform to `APIResponseEntity<T>` (`status`, `statusCode`, `message`, `data`).
- [ ] Structured Pino logger outputs single-line JSON records containing `correlationId`.
- [ ] Controllers contain zero imports from repositories or ORM query builders.
- [ ] All database-specific operations sit behind explicit repository interfaces.

---

## 5. Configuration & Environment Security
- [ ] Startup configuration validates fail-fast using typed schema before opening server ports.
- [ ] Dedicated `.env.example` file committed in each service root documenting all required keys.
- [ ] Zero production secrets, private keys, or passwords committed to git.
- [ ] `.env` and `.env.*` excluded in `.gitignore`.

---

## 6. Provenance & Agent Artifacts
- [ ] Target project root `README.md` stamped with: `Scaffolded using AI-Ready Project Standards v0.3.0`.
- [ ] Project-level `AGENTS.md` generated inside target repository documenting its scripts, directory layout, and architectural conventions.
- [ ] Conformance audit completed and reported to the user.
