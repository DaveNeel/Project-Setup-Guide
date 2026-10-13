# Shared Code Standard: Monolith Common Folder (Tier 1)

## 1. Purpose
Defines directory structure, encapsulation rules, and unidirectional dependency constraints for shared code within a monolithic application.

---

## 2. Applies when
- `architecture: monolith` or `modular-monolith` is selected in Step 1.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Common Folder |
|:---|:---|
| Step 1 (Architecture) | Automatically selected. Monorepo packages are omitted. |
| Step 2 (Framework) | Houses framework-specific filters, interceptors, and middleware. |

---

## 4. Contract

### Directory Organization
- All shared application cross-cutting code MUST reside under `src/common/`.
- Subdirectories MUST be strictly organized by architectural concern:
  - `src/common/errors/`: Custom error classes extending `AppError`.
  - `src/common/filters/`: Exception filters (NestJS).
  - `src/common/interceptors/`: Response interceptors (NestJS).
  - `src/common/middleware/`: Express middleware / NestJS middleware.
  - `src/common/guards/`: Authentication and role guards.
  - `src/common/logger/`: Structured logging abstractions.
  - `src/common/utils/`: Generic utility helper functions.

### Strict Unidirectional Dependency Rule
- Code inside `src/common/` MUST NEVER import from `src/modules/` or feature domains.
- Feature modules (`src/modules/*`) MAY import from `src/common/`.
- Any circular dependency between `src/common/` and `src/modules/` is STRICTLY PROHIBITED.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// src/common/errors/app.error.ts
// CORRECT: Base error has zero dependencies on domain modules
export abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly errorCode: string;

  constructor(message: string, public readonly details?: unknown) {
    super(message);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}
```

---

## 6. Agent Output Checklist
- [ ] `src/common/` directory created with designated subfolders.
- [ ] Zero imports from `src/modules/` inside `src/common/`.
- [ ] Base error classes and envelope formatters reside in `src/common/`.

---

## 7. Common Mistakes
- **Circular Imports:** Importing `UserService` or `UserEntity` into `src/common/guards/auth.guard.ts`.
- **Dumping Domain Logic in Common:** Putting domain-specific calculations into `src/common/utils/`.
