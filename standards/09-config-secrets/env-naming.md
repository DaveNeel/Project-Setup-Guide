# Config & Secrets Standard: Environment Naming & .env.example (Tier 1)

## 1. Purpose
Defines syntax rules, canonical variable names, and security constraints for environment configuration files.

---

## 2. Applies when
- All projects across all architectural topologies.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Monolith | Single `.env.example` file located at project root. |
| Microservices | Dedicated `.env.example` file located within each service directory (`services/*/.env.example`). |

---

## 4. Contract

### Naming Conventions & Casing
- All environment variable keys MUST be uppercase snake_case: `UPPER_SNAKE_CASE`.
- Canonical standard variable keys:
  - `NODE_ENV`: Allowed values: `development`, `production`, `test`.
  - `PORT`: TCP port number (e.g. `3000`).
  - `DATABASE_URL` / `MONGODB_URI`: Canonical database connection string.
  - `JWT_SECRET`: Secret key for JWT signature verification (minimum 32 characters).
  - `JWT_EXPIRES_IN`: Token validity duration (e.g. `1h`, `86400s`).
  - `LOG_LEVEL`: Pino log level: `debug`, `info`, `warn`, `error` (default: `info`).
  - `CORS_ORIGIN`: Allowed origins separated by comma (default: `*` in local dev).

### `.env.example` & Git Security Rules
- Every deployable service MUST contain a `.env.example` file committed to version control.
- `.env.example` MUST list EVERY key required by the service with a documented dummy value.
- Real credentials, passwords, private keys, or API tokens MUST NEVER be committed to git.
- `.env`, `.env.local`, and `.env.*.local` MUST be explicitly ignored in `.gitignore`.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```ini
# .env.example canonical template
NODE_ENV=development
PORT=3000
LOG_LEVEL=info
CORS_ORIGIN=http://localhost:3000

# Database
DATABASE_URL=postgresql://postgres:postgres_password@localhost:5432/app_db

# Authentication
JWT_SECRET=super_secret_jwt_key_at_least_32_chars_long
JWT_EXPIRES_IN=1h
```

---

## 6. Agent Output Checklist
- [ ] `.env.example` created with all required variables.
- [ ] Keys use strict UPPER_SNAKE_CASE.
- [ ] Root `.gitignore` includes `.env` and `.env.*`.
- [ ] No real secrets or production passwords in template files.

---

## 7. Common Mistakes
- **Committing Real Secrets:** Writing real database passwords or live API keys in `.env.example`.
- **Inconsistent Casing:** Using camelCase like `databaseUrl` or lowercase `port`.
- **Missing Keys in `.env.example`:** Adding a new variable in code without updating `.env.example`.
