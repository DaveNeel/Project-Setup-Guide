# Config & Secrets Standard: Fail-Fast Typed Config (Tier 1)

## 1. Purpose
Defines contracts for runtime configuration validation, schema enforcement, and typed configuration injection to guarantee that invalid environments fail fast at startup.

---

## 2. Applies when
- `config_strategy: typed-config` is selected in Step 9.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Config Schema |
|:---|:---|
| Step 2 (Framework) | NestJS: `@nestjs/config` with `validate` hook.<br>Express: startup validation function in `src/config/env.config.ts`. |
| Step 3 (Database Engine) | Injects database-specific connection keys (`DATABASE_URL`, `DB_PORT`, etc.). |

---

## 4. Contract

### Startup Validation (Fail-Fast)
- Configuration validation MUST execute as the very first step of application bootstrap, BEFORE opening database connections or starting the HTTP server.
- The schema MUST validate data types, allowed enums (e.g. `NODE_ENV: 'development' | 'production' | 'test'`), port ranges, and required string formats.
- If any required environment variable is missing or invalid, the process MUST print a descriptive error listing the invalid keys and terminate immediately (`process.exit(1)`).

### Strict Prohibition of `process.env`
- Direct access to `process.env` outside of the `src/config/` directory is STRICTLY PROHIBITED.
- Application code, services, and repositories MUST inject or import the typed configuration object or service.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Fail-fast schema validation contract (Zod example)
const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32)
});

export function validateEnv(rawEnv: Record<string, unknown>) {
  const result = EnvSchema.safeParse(rawEnv);
  if (!result.success) {
    console.error('Fatal Config Error:', result.error.format());
    process.exit(1);
  }
  return result.data;
}
```

---

## 6. Agent Output Checklist
- [ ] Schema validation file created in `src/config/`.
- [ ] Bootstrap calls validation before initializing services.
- [ ] Zero references to `process.env` in controllers, services, or repositories.

---

## 7. Common Mistakes
- **Late Discovery of Missing Variables:** Booting the app and only failing minutes later when an unconfigured endpoint is invoked.
- **Reading `process.env` in Business Logic:** Injecting raw environment strings into domain services instead of typed config properties.
