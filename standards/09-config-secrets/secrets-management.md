# Config & Secrets Standard: Secrets Management (Tier 1, Tier 2 Delta & Tier 3)

## 1. Purpose
Defines external secret resolution, provider integration, and cloud secrets retrieval patterns.

---

## 2. Applies when
- Evaluated during Step 9 (`config_strategy`).

---

## 3. Interaction Notes

| Strategy | Support Tier | Status |
|:---|:---:|:---|
| `typed-config` (env) | Tier 1 | Base standard in `typed-config.md`. |
| `aws-secrets-manager` | Tier 2 | AWS Secrets Manager retrieval delta. |
| `vault` | Tier 3 | **NOT YET SUPPORTED** banner. |

---

## 4. Contract

### Option A: `typed-config` (Tier 1 Base)
- Secrets are supplied via standard operating system environment variables or local `.env` files.
- Application parses and validates them using `standards/09-config-secrets/typed-config.md`.

### Option B: `aws-secrets-manager` (Tier 2 Delta)
Identical to `standards/09-config-secrets/typed-config.md`, **EXCEPT**:
- At application startup, an async bootstrap pre-hook retrieves secret key-value pairs from AWS Secrets Manager using `@aws-sdk/client-secrets-manager`.
- The secret name is determined by `AWS_SECRET_NAME` and `AWS_REGION`.
- Retrieved secrets are merged with process environment variables BEFORE schema validation.
- Secrets are cached in memory for the lifecycle of the process; requests MUST NOT make runtime network calls to AWS Secrets Manager.

### Option C: `vault` (Tier 3 — Not Supported)
```
====================================================================
[NOT YET SUPPORTED]
HashiCorp Vault secret integration is currently classified under Tier 3.
The standards repository does not yet define certified Vault token
lifecycle, AppRole authentication, or dynamic secret lease renewal.
====================================================================
```

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// AWS Secrets Manager bootstrap hook
export async function loadAwsSecrets(secretName: string, region: string): Promise<Record<string, string>> {
  const client = new SecretsManagerClient({ region });
  const response = await client.send(new GetSecretValueCommand({ SecretId: secretName }));
  return response.SecretString ? JSON.parse(response.SecretString) : {};
}
```

---

## 6. Agent Output Checklist
- [ ] If `vault` chosen, agent halts and outputs Tier 3 banner.
- [ ] If `aws-secrets-manager` chosen, pre-bootstrap loader implemented with in-memory caching.
- [ ] Validated configuration still runs against the unified schema.

---

## 7. Common Mistakes
- **Fetching Secrets Per Request:** Calling AWS Secrets Manager on every incoming HTTP request.
- **Bypassing Local Schema Validation:** Injecting AWS secrets directly without running schema parsing.
