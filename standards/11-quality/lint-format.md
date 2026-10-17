# Quality Standard: Linting, Formatting & Strict TypeScript (Tier 1)

## 1. Purpose
Defines mandatory rules for static analysis, automated formatting, and strict TypeScript compiler settings across all generated projects.

---

## 2. Applies when
- All projects across all architectural topologies.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Monolith | Single root `tsconfig.json`, `.prettierrc`, and `eslint.config.mjs`. |
| Microservices | Root `tsconfig.base.json` extended by individual `services/*/tsconfig.json`. |

---

## 4. Contract

### TypeScript Strictness Contract
All `tsconfig.json` files MUST configure:
- `"target": "ES2022"`
- `"strict": true`
- `"noImplicitAny": true`
- `"strictNullChecks": true`
- `"noUnusedLocals": true`
- `"noUnusedParameters": true`
- `"exactOptionalPropertyTypes": true`
- `"skipLibCheck": true`
- `"sourceMap": false` (prohibits generation of `.js.map` files)
- `"declarationMap": false` (prohibits generation of `.d.ts.map` files in `shared-utils`)

### Linting & Formatting Contract
- Linting MUST use ESLint flat config (`eslint.config.mjs`) with `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin`.
- Formatting MUST use Prettier (`.prettierrc`) with:
  - `{ "singleQuote": true, "trailingComma": "all", "printWidth": 100, "tabWidth": 2, "semi": true }`
- `package.json` MUST expose:
  - `"lint"`: Runs ESLint checks.
  - `"format"`: Runs Prettier check/write.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```json
// .prettierrc canonical configuration
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "semi": true
}
```

---

## 6. Agent Output Checklist
- [ ] Root `eslint.config.mjs` and `.prettierrc` created.
- [ ] TypeScript config enforces `strict: true` and `exactOptionalPropertyTypes: true`.
- [ ] `npm run lint` executes and exits with code 0.

---

## 7. Common Mistakes
- **Disabling Strict Mode:** Setting `"strict": false` to bypass typing errors.
- **Using Legacy `.eslintrc`:** Using deprecated ESLint configurations instead of modern flat config (`eslint.config.mjs`).
- **Ignoring Unused Variables:** Leaving unused function parameters without an underscore prefix (`_req`).
