# Authentication Standard: No Authentication (Tier 3 — Not Supported)

```
====================================================================
[NOT YET SUPPORTED]
Disabling authentication ('none') is classified under Tier 3.
All production backends scaffolded under these standards require
authenticated identities by default.
====================================================================
```

## 1. Purpose
Documents the support tier status of unauthenticated APIs.

---

## 2. Applies when
- `auth: none` is selected in Step 10.

---

## 3. Mandatory Agent Action
- The AI agent MUST NOT scaffold completely unauthenticated backends without explicit human override.
- The agent MUST halt, display the banner above, and ask the user to select the active Tier 1 standard: `jwt`.
