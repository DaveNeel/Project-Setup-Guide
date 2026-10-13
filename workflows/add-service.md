# Workflow: Add Microservice

Numbered procedure for adding a new service to an existing microservices project scaffolded under these standards.

---

## 1. Specification & Port Allocation
1. Define the service name: lowercase kebab-case ending in `-service` (e.g. `notification-service`).
2. Define the single bounded context responsibility.
3. Determine database ownership (`own_db: true` or `false`).
4. Allocate a unique dedicated HTTP port not used by existing services (e.g. `3004`).

---

## 2. Directory & Manifest Scaffolding
1. Create `services/<service-name>/`.
2. Scaffold `services/<service-name>/package.json` extending root workspace packages:
   - Include dependency: `"shared-utils": "workspace:*"`.
   - Include standard scripts: `"build"`, `"lint"`, `"start:dev"`.
3. Scaffold `tsconfig.json` extending root `tsconfig.base.json`.
4. Create `.env.example` defining `PORT`, `NODE_ENV`, `LOG_LEVEL`, and database URLs if applicable.

---

## 3. Core Component Integration
1. Scaffold `src/main.ts` (or `src/server.ts`) implementing the standard bootstrap sequence.
2. Implement `src/config/` with startup schema validation using `NodeEnvEnum` and `LogLevelEnum`.
3. Mount correlation ID middleware, structured Pino logger, and global exception filter/middleware.
4. Mount single health check probe endpoint at `GET /health` performing active DB connectivity verification.

---

## 4. Persistence Layer (If `own_db: true`)
1. Implement database connection and standalone `data-source.ts`.
2. Create `src/database/migrations/` and add migration scripts to `package.json`.
3. Scaffold domain entities and repository classes implementing `I*Repository` interfaces.

---

## 5. Domain Logic & Endpoint Implementation
1. Scaffold feature module with DTOs, controller, and domain service.
2. In Express, declare routes via `RouteDefinition[]` and mount using `registerRoutes(router, routes)`.
3. Wrap controller actions via `handleRequestWithResponse`.
4. Ensure endpoints use `/api/v1/` prefix and return standard envelopes with enums.

---

## 6. Baseline Verification
1. Run from workspace root:
   - `npm install`
   - `npm run build --filter=<service-name>`
   - `npm run lint --filter=<service-name>`
2. Verify zero errors before declaring service addition complete.
