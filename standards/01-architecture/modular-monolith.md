# Architecture Standard: Modular Monolith (Tier 2 Delta)

## 1. Purpose
Defines the architectural delta for scaffolding a single deployable application with strictly enforced internal domain boundaries.

---

## 2. Applies when
- `architecture: modular-monolith` is selected in Step 1.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Base Standard | Inherits all rules from `standards/01-architecture/monolith.md`. |
| Steps 5, 6, 7 | Automatically marked **SKIPPED** (same as monolith). |
| Step 8 | Locked to `monolith-common-folder` (`src/common/`). |

---

## 4. Contract (Delta from Monolith)

This standard is identical to `standards/01-architecture/monolith.md`, **EXCEPT**:

1. **Strict Public Domain Facades:**
   - Every module under `src/modules/<domain>/` MUST expose a public interface or facade through an `index.ts` file located at the module root (`src/modules/<domain>/index.ts`).
   - Private module internals (entities, internal services, repositories) MUST NOT be exported from the module's `index.ts`.
2. **Prohibited Cross-Module Repository Access:**
   - A module MUST NOT import or query a repository, entity, or data-access model belonging to another module.
   - If Module A requires data or operations from Module B, it MUST call Module B's exported public service or interface.
3. **Decoupled Module Events:**
   - Modules SHOULD communicate asynchronous domain side-effects using an in-process EventEmitter (e.g. Node.js `EventEmitter` or `@nestjs/event-emitter`), avoiding direct synchronous coupling where possible.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// src/modules/order/order.service.ts
// CORRECT: Interacting via public UserService interface, NOT UserRepository
export class OrderService {
  constructor(private readonly userFacade: UserFacade) {}

  async checkout(userId: string) {
    const user = await this.userFacade.getUserSummary(userId);
    // Proceed with checkout logic...
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Each domain module has an `index.ts` exporting only public services and DTOs.
- [ ] Zero cross-module imports into private repository files (`../user/user.repository` inside `order` is rejected).
- [ ] In-process event emitter configured for cross-domain side-effects.

---

## 7. Common Mistakes
- **Direct Repository Leaks:** Injecting `UserRepository` directly into `OrderService`.
- **Circular Module Dependencies:** Having Module A import Module B while Module B imports Module A.
