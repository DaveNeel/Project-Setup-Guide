# Quality Standard: Testing Baseline (Deferred for Setup)

## 1. Purpose
Defines testing guidelines and establishes that initial baseline project setup intentionally defers test folder scaffolding.

---

## 2. Applies when
- All project setups.

---

## 3. Interaction Notes

| Scope | Rule |
|:---|:---|
| Baseline Setup | Test folders (`test/`) and test suites are NOT generated during initial project setup. |
| Future Expansion | Unit and integration test suites can be introduced incrementally as feature logic matures. |

---

## 4. Contract

### Deferred Test Scaffolding
- The initial project setup MUST focus exclusively on clean, compilable, and runnable service source code (`src/`).
- The AI agent MUST NOT generate `test/` folders, `*.spec.ts`, or `*.test.ts` files during initial setup.
- Verification during initial setup is executed via `npm run build` (TypeScript compilation) and `npm run lint` (static analysis).

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```
Target Service Setup Tree:
├── package.json
├── tsconfig.json
├── .env.example
└── src/                     # Source code only (no test/ folder in baseline)
    ├── server.ts
    ├── app.ts
    └── modules/
```

---

## 6. Agent Output Checklist
- [ ] No `test/` directories scaffolded in services during baseline setup.
- [ ] Build script compiles cleanly with zero TypeScript errors.

---

## 7. Common Mistakes
- **Scaffolding Unnecessary Test Folders:** Adding boilerplate tests to a basic project setup before feature requirements are defined.
