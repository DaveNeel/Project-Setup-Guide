# Data Access Standard: Mongoose (Tier 1)

## 1. Purpose
Defines contracts for schema declarations, document models, and repository implementations using Mongoose for MongoDB.

---

## 2. Applies when
- `data_access: mongoose` is selected in Step 4.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Mongoose |
|:---|:---|
| Step 3 (Database Engine) | Locked to `mongodb`. SQL engines are rejected. |
| Step 2 (Framework) | NestJS: `@nestjs/mongoose`<br>Express: direct `mongoose.connect` lifecycle |

---

## 4. Contract

### Documentation Verification Rule
> [!IMPORTANT]
> The AI agent MUST verify current official Mongoose documentation at generation time for schema definitions, connection hooks, and model compilation.

### Schema & Model Locations
- Schemas MUST reside in: `src/modules/<domain>/schemas/<entity>.schema.ts`.
- Models MUST be compiled with strict schema options:
  - `{ timestamps: true, strict: true, versionKey: false }`
- Collection names MUST be explicitly defined in lowercase plural: `mongoose.model('User', UserSchema, 'users')`.

### Repository Wrapper
- Concrete repositories MUST implement `standards/04-data-access/repository-pattern.md`.
- Repositories wrap Mongoose's `Model<T>` methods (`findById`, `create`, `updateOne`).
- Mongoose documents MUST be converted to plain JavaScript objects (`.lean()` or domain mapper) before returning across the repository boundary.
- Mongoose `_id` MUST be converted to string `id` in mapped domain entities.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Mongoose repository implementation wrapper
@Injectable()
export class MongooseUserRepository implements IUserRepository {
  constructor(@InjectModel(User.name) private readonly userModel: Model<UserDocument>) {}

  async findById(id: string): Promise<UserDomain | null> {
    const doc = await this.userModel.findById(id).lean().exec();
    return doc ? this.toDomain(doc) : null;
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Schemas located under `src/modules/<domain>/schemas/`.
- [ ] Repository implementations use `.lean()` or domain mappers to decouple Mongoose documents.
- [ ] Strict schema and timestamps options enabled.
- [ ] No direct Mongoose Model imports in domain services.

---

## 7. Common Mistakes
- **Returning Mongoose Hydrated Documents:** Passing active documents with internal methods to services/controllers.
- **Missing `versionKey: false` or Serialization:** Allowing internal `__v` to escape into responses.
- **Unindexed Queries:** Querying unique fields like email without declaring `{ unique: true }`.
