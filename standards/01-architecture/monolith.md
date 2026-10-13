# Architecture Standard: Monolith (Tier 1)

## 1. Purpose
Defines the architectural contract for scaffolding a single, unified deployable backend application.

---

## 2. Applies when
- `architecture: monolith` is selected in Step 1.

---

## 3. Interaction Notes

| Decision | Impact on Monolith |
|:---|:---|
| Steps 5, 6, 7 | Automatically marked **SKIPPED**. Monolith has one database and in-process execution. |
| Step 8 (Shared Code) | Automatically locked to `monolith-common-folder` (`src/common/`). Workspace packages are not created. |

---

## 4. Contract

### Repository & Application Structure
- The repository root MUST house a single deployable application with one root `package.json` and `tsconfig.json`.
- The application source code MUST reside under `src/`.
- Domain features MUST be organized under `src/modules/<domain>/` or `src/features/<domain>/`.
- Shared cross-cutting logic MUST reside under `src/common/`.
- The application MUST expose a single entry point:
  - For NestJS: `src/main.ts`
  - For Express: `src/server.ts`
- The application MUST bind to a single configurable HTTP port defined by `PORT` (default: 3000).
- The application MUST manage a single database connection pool initialized at application startup.
- Cross-domain interactions MUST occur through direct dependency injection or internal service calls, NEVER via loopback HTTP requests.

---

## 5. Illustrative Tree
<!-- illustrative only -->
```
my-monolith-app/
├── package.json              # Single application manifest
├── tsconfig.json             # Root TypeScript configuration
├── src/
│   ├── common/               # Shared filters, guards, interceptors, utils
│   ├── config/               # Validated configuration module
│   ├── database/             # Database connection & migration setup
│   ├── modules/
│   │   ├── auth/             # Authentication domain
│   │   └── user/             # User domain
│   └── main.ts               # Unified application entry point
```

---

## 6. Agent Output Checklist
- [ ] Single root `package.json` with scripts: `build`, `lint`, `test`, `start:dev`.
- [ ] Source tree rooted at `src/` containing `common/`, `config/`, and domain modules under `modules/`.
- [ ] Exactly one application bootstrap entry point (`main.ts` or `server.ts`).
- [ ] No monorepo `packages/` or `services/` directories present.

---

## 7. Common Mistakes
- **Creating Monorepo Workspaces:** Generating `workspaces: []` in `package.json` or creating a `services/` directory.
- **Loopback HTTP Calls:** Making HTTP requests to `localhost:3000` to interact between internal domain modules.
- **Scattering Common Utilities:** Creating disconnected `utils/` or `helpers/` folders instead of consolidating under `src/common/`.
