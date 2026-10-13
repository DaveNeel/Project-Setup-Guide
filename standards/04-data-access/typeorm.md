# Data Access Standard: TypeORM (Tier 1)

## 1. Purpose
Defines contracts for entity declarations, migrations, connection configuration, and repository implementations using TypeORM.

---

## 2. Applies when
- `data_access: typeorm` is selected in Step 4.

---

## 3. Interaction Notes

| Preceding Decision | Impact on TypeORM |
|:---|:---|
| Step 3 (Database Engine) | `postgres`: driver `pg`<br>`mysql`: driver `mysql2`<br>`mssql`: driver `mssql` |
| Step 2 (Framework) | NestJS: `@nestjs/typeorm`<br>Express: standalone `DataSource` instance |

---

## 4. Contract

### Documentation Verification Rule
> [!IMPORTANT]
> The AI agent MUST verify current official TypeORM documentation at generation time for engine driver versions, syntax, and migration CLI commands before scaffolding.

### Entity Locations & Naming
- Entities MUST reside in: `src/modules/<domain>/entities/<entity>.entity.ts`.
- Entity classes MUST be named with PascalCase ending in `Entity` (e.g. `UserEntity`).
- Database table names MUST be explicit via `@Entity({ name: 'users' })`.
- Columns MUST explicitly declare database type and `name: 'column_name'` in snake_case.

### Migrations
- Migration files MUST reside in: `src/database/migrations/`.
- Migrations MUST be compiled and executed using an explicit `DataSource` config file: `src/database/data-source.ts`.
- `package.json` MUST define standard migration scripts:
  - `"migration:generate"`
  - `"migration:run"`
  - `"migration:revert"`
- `synchronize: false` MUST be enforced in production environments.

### Repository Wrapper
- Concrete repositories MUST implement `standards/04-data-access/repository-pattern.md`.
- Repositories wrap TypeORM's `Repository<T>` and convert between `TEntity` and plain domain models.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// TypeORM DataSource contract
export const AppDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [UserEntity],
  migrations: ['src/database/migrations/*.ts'],
  synchronize: false,
  logging: process.env.NODE_ENV === 'development'
});
```

---

## 6. Agent Output Checklist
- [ ] Explicit `data-source.ts` file configured with `synchronize: false`.
- [ ] Entities located in `src/modules/<domain>/entities/*.entity.ts`.
- [ ] Migration scripts wired into `package.json`.
- [ ] Repository implementations encapsulate TypeORM calls.

---

## 7. Common Mistakes
- **Missing `data-source.ts`:** Omitting standalone DataSource, causing TypeORM CLI migration commands to fail.
- **Enabling `synchronize: true`:** Dropping or altering production schemas at runtime.
- **Forgetting Snake_Case Columns:** Neglecting `{ name: 'created_at' }` on `@CreateDateColumn()`.
