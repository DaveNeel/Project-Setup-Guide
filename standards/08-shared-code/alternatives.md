# Shared Code Standard: Alternatives (Tier 2 Delta & Tier 3)

## 1. Purpose
Defines secondary strategies for sharing code across services: Git Submodules (Tier 2), Published Private Packages (Tier 2), and None (Tier 3).

---

## 2. Applies when
- `shared_code_strategy: git-submodule | published-private-packages | none` is selected in Step 8.

---

## 3. Interaction Notes

| Strategy | Support Tier | Status |
|:---|:---:|:---|
| `shared-utils` | Tier 1 | Full standard in `workspace-packages.md`. |
| `git-submodule` | Tier 2 | Pinned Git submodule delta. |
| `published-private-packages` | Tier 2 | Private npm registry package delta. |
| `none` | Tier 3 | **NOT YET SUPPORTED** banner. |

---

## 4. Contract

### Option A: `git-submodule` (Tier 2 Delta)
Identical to `standards/08-shared-code/workspace-packages.md`, **EXCEPT**:
- `packages/shared-utils/` is managed as an independent Git repository tracked via `.gitmodules`.
- Root project MUST document submodule clone initialization: `git submodule update --init --recursive`.
- Services import via local filesystem references.

### Option B: `published-private-packages` (Tier 2 Delta)
Identical to `standards/08-shared-code/workspace-packages.md`, **EXCEPT**:
- Shared code is published to a private registry (e.g. GitHub Packages, private npm).
- Services declare explicit semver dependencies: `"@company/shared-utils": "^1.0.0"`.
- Root `.npmrc` MUST document registry authentication requirements.

### Option C: `none` (Tier 3 — Not Supported)
```
====================================================================
[NOT YET SUPPORTED]
Operating microservices with zero shared code contracts is Tier 3.
The AI agent MUST stop and prompt the user to select an active
sharing strategy (shared-utils [T1], git-submodule [T2]).
====================================================================
```

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```ini
# .gitmodules illustration for Tier 2 git-submodule
[submodule "packages/shared-utils"]
	path = packages/shared-utils
	url = git@github.com:organization/shared-utils.git
```

---

## 6. Agent Output Checklist
- [ ] If `none` chosen, agent halts and outputs Tier 3 banner.
- [ ] If `git-submodule` chosen, `.gitmodules` template provided.
- [ ] If `published-private-packages` chosen, `.npmrc.example` template provided.

---

## 7. Common Mistakes
- **Allowing `none` Silently:** Copy-pasting boilerplate across services without common interfaces.
