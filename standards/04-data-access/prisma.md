# Data Access Standard: Prisma (Tier 2 Delta)

## 1. Purpose
Defines the configuration and pattern delta for using Prisma ORM relative to the TypeORM Tier 1 standard.

---

## 2. Applies when
- `data_access: prisma` is selected in Step 4.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Prisma |
|:---|:---|
| Base Standard | Inherits repository boundaries from `standards/04-data-access/repository-pattern.md` and `typeorm.md`. |
| Step 3 (Database Engine) | Configures datasource provider in `schema.prisma` (`postgresql`, `mysql`, `sqlserver`). |

---

## 4. Contract (Delta from TypeORM)

This standard is identical to `standards/04-data-access/typeorm.md`, **EXCEPT**:

1. **Verification Rule:**
   - The AI agent MUST verify current official Prisma documentation at generation time for schema syntax, generator settings, and migration workflows.
2. **Schema Location:**
   - Prisma models are NOT declared as TypeScript classes in `entities/`. All models MUST reside in a single declarative schema file: `prisma/schema.prisma`.
3. **Migration Commands:**
   - Migration files reside in `prisma/migrations/`.
   - `package.json` scripts MUST define:
     - `"db:migrate:dev": "prisma migrate dev"`
     - `"db:migrate:deploy": "prisma migrate deploy"`
     - `"db:generate": "prisma generate"`
4. **Repository Implementation:**
   - Repositories wrap `PrismaClient` (or a subclass `PrismaService`).
   - Domain services MUST NOT import `PrismaClient` directly; they call repository interfaces.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Prisma repository wrapper contract
@Injectable()
export class PrismaUserRepository implements IUserRepository {
  constructor(private readonly prisma: PrismaService) {}

  async findById(id: string): Promise<UserDomain | null> {
    const record = await this.prisma.user.findUnique({ where: { id } });
    return record ? this.toDomain(record) : null;
  }
}
```

---

## 6. Agent Output Checklist
- [ ] `prisma/schema.prisma` configured with correct database provider and client generator.
- [ ] `package.json` includes `prisma generate` in build step.
- [ ] Repositories wrap `PrismaClient` queries.

---

## 7. Common Mistakes
- **Failing to Run `prisma generate` in Build:** Forgetting postinstall/build hook, resulting in missing client types.
- **Importing Prisma Types in Controllers:** Leaking Prisma-generated model types across domain boundaries.
