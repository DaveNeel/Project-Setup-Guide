# Database Standard: NoSQL Common (Document)

## 1. Purpose
Defines shared database contracts for document-based NoSQL engines, establishing strict schema enforcement, index declarations, and lifecycle management.

---

## 2. Applies when
- `database_engine: mongodb` is selected in Step 3.

---

## 3. Interaction Notes

| Dimension | Rule |
|:---|:---|
| Step 4 (Data Access) | Implemented exclusively via `mongoose` (Tier 1). Relational ORMs are rejected. |
| Step 5 (DB Strategy) | Dictates whether services share a MongoDB cluster or maintain dedicated databases. |

---

## 4. Contract

### Schema Enforcement & Sanitization
- Although the underlying engine is schemaless, applications MUST enforce strict schemas at the application layer via model definitions.
- Schema definitions MUST specify `timestamps: true` (creating `createdAt` and `updatedAt` in UTC).
- Field properties MUST specify explicit types, required flags, and defaults where appropriate.
- Schemas MUST enable `{ strict: true }` to reject unmapped fields.

### Connection Lifecycle & Indexing
- Database connection URI MUST be validated via typed configuration schema.
- Automatic runtime index creation (`autoIndex`) MUST be disabled in production environments (`autoIndex: false`) to avoid startup latency; indexes MUST be created via explicit maintenance scripts.
- Connections MUST configure connection pool limits: `minPoolSize` and `maxPoolSize` via environment variables.
- Shutdown hooks MUST cleanly disconnect via `connection.close()` upon `SIGINT` / `SIGTERM`.

### Document Naming Conventions
- Collections MUST be plural and lowercase (e.g. `users`, `orders`).
- Document IDs MUST be mapped to a clean string representation `id` in external API envelopes, converting internal `_id` and stripping internal version keys (`__v`).

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Mongoose schema transform contract
export const UserSchema = new Schema({
  email: { type: String, required: true, unique: true, lowercase: true, trim: true },
  name: { type: String, required: true }
}, {
  timestamps: true,
  strict: true,
  toJSON: {
    transform: (_, ret) => {
      ret.id = ret._id.toString();
      delete ret._id;
      delete ret.__v;
    }
  }
});
```

---

## 6. Agent Output Checklist
- [ ] Mongoose schemas configured with `timestamps: true` and `strict: true`.
- [ ] External serialization transforms `_id` to string `id` and strips `__v`.
- [ ] Production connection settings disable `autoIndex`.

---

## 7. Common Mistakes
- **Leaking `_id` and `__v`:** Exposing raw MongoDB internals in public API response envelopes.
- **Enabling autoIndex in Production:** Allowing Mongoose to build heavy compound indexes on every process start.
- **Ignoring Validation Errors:** Failing to translate Mongoose `ValidationError` into standard 400 Bad Request envelopes.
