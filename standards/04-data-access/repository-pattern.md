# Data Access Standard: Repository Pattern Boundary

## 1. Purpose
Defines the mandatory repository abstraction boundary ensuring business services remain completely agnostic of underlying ORM/ODM implementations.

---

## 2. Applies when
- All projects across all architectural topologies and frameworks.

---

## 3. Interaction Notes

| Layer | Dependency Direction |
|:---|:---|
| Controllers | Call Services. MUST NOT import Repositories or ORM Entities. |
| Services | Call Repositories via Interfaces. MUST NOT import ORM Query Builders or Models directly. |
| Repositories | Import ORM entities/models. Encapsulate all database persistence operations. |

---

## 4. Contract

### Repository Abstraction Rules
- Every persisted aggregate or entity MUST have a corresponding repository interface and implementation:
  - Interface: `src/modules/<domain>/repositories/<entity>.repository.interface.ts`
  - Implementation: `src/modules/<domain>/repositories/<entity>.repository.ts`
- Domain services MUST depend exclusively on the repository interface (or injected repository token), NEVER on the raw ORM/ODM context.
- The repository implementation MUST encapsulate all ORM-specific operations:
  - TypeORM: `Repository<T>` calls, `createQueryBuilder()`.
  - Mongoose: `Model<T>` calls, `.find()`, `.aggregate()`.
  - Prisma: `prismaClient.<model>` calls.
  - Drizzle: `db.select()`, `db.insert()`.
- Controllers MUST NOT import or inject repository classes directly.
- Domain methods MUST receive and return plain domain entities or DTOs, stripping ORM-specific metadata before returning to business services.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Strict repository boundary illustration
export interface IUserRepository {
  findById(id: string): Promise<UserDomain | null>;
  create(data: CreateUserData): Promise<UserDomain>;
}

@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(@InjectRepository(UserEntity) private readonly repo: Repository<UserEntity>) {}

  async findById(id: string): Promise<UserDomain | null> {
    const entity = await this.repo.findOneBy({ id });
    return entity ? this.toDomain(entity) : null;
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Explicit repository interface file (`*.repository.interface.ts`) created for each aggregate.
- [ ] Concrete repository implementation (`*.repository.ts`) implements the interface.
- [ ] Domain service injects the repository abstraction, with zero ORM imports in the service.
- [ ] Controllers contain zero repository references.

---

## 7. Common Mistakes
- **Direct ORM Imports in Services:** Calling `this.userRepo.createQueryBuilder()` inside `UserService`.
- **Controllers Injecting Repositories:** Bypassing the service layer by calling repository methods directly inside controller route handlers.
- **Leaking ORM Entity Instances:** Returning mutable TypeORM/Mongoose entities containing active query runners directly to HTTP responses.
