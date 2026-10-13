# Data Access Standard: Drizzle (Tier 2 Delta)

## 1. Purpose
Defines the configuration and pattern delta for using Drizzle ORM relative to the TypeORM Tier 1 standard.

---

## 2. Applies when
- `data_access: drizzle` is selected in Step 4.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Drizzle |
|:---|:---|
| Base Standard | Inherits repository boundaries from `standards/04-data-access/repository-pattern.md` and `typeorm.md`. |
| Step 3 (Database Engine) | `postgres`: `drizzle-orm/node-postgres`<br>`mysql`: `drizzle-orm/mysql2`<br>`mssql`: `drizzle-orm/node-postgres` or custom dialect |

---

## 4. Contract (Delta from TypeORM)

This standard is identical to `standards/04-data-access/typeorm.md`, **EXCEPT**:

1. **Verification Rule:**
   - The AI agent MUST verify current official Drizzle documentation at generation time for schema declarations, driver adapters, and `drizzle-kit` commands.
2. **Schema Locations:**
   - Schemas are declared using TypeScript table helpers (`pgTable`, `mysqlTable`) in: `src/database/schema/<domain>.schema.ts`.
   - Table and column names MUST be lowercase `snake_case`.
3. **Migration & Kit Configuration:**
   - Root configuration file MUST be: `drizzle.config.ts`.
   - Migration files reside in: `drizzle/migrations/`.
   - `package.json` scripts MUST define:
     - `"db:generate": "drizzle-kit generate"`
     - `"db:migrate": "drizzle-kit migrate"`
4. **Repository Implementation:**
   - Repositories wrap the typed Drizzle database client instance (`NodePgDatabase`, etc.).
   - Domain services call repository methods without directly importing Drizzle table schemas or query builders.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Drizzle repository wrapper contract
@Injectable()
export class DrizzleUserRepository implements IUserRepository {
  constructor(@Inject('DRIZZLE_DB') private readonly db: NodePgDatabase) {}

  async findById(id: string): Promise<UserDomain | null> {
    const records = await this.db.select().from(users).where(eq(users.id, id)).limit(1);
    return records[0] ? this.toDomain(records[0]) : null;
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Root `drizzle.config.ts` configured with dialect and schema paths.
- [ ] Schema files located under `src/database/schema/`.
- [ ] Repositories wrap Drizzle queries behind interface contracts.

---

## 7. Common Mistakes
- **Missing `drizzle.config.ts`:** Omitting kit configuration prevents migration generation.
- **Exporting Raw DB Client to Controllers:** Permitting controllers to call `db.select()` directly.
