# Authentication Standard: Session-Based Auth (Tier 3 — Not Supported)

```
====================================================================
[NOT YET SUPPORTED]
Stateful session-based cookie authentication is classified under Tier 3.
The standards repository does not currently define certified Redis
session stores, cookie serialization, or CSRF token handling contracts.
====================================================================
```

## 1. Purpose
Documents the support tier status of stateful cookie/session authentication.

---

## 2. Applies when
- `auth: session` is selected in Step 10.

---

## 3. Mandatory Agent Action
- The AI agent MUST NOT generate stateful session implementations.
- The agent MUST halt, display the banner above, and ask the user to select the active Tier 1 standard: `jwt`.
