# Service Standard: Service Boundaries & Domain Decomposition (Tier 1)

## 1. Purpose
Defines principles, naming rules, and anti-corruption patterns governing how microservice responsibilities and boundaries are established.

---

## 2. Applies when
- `architecture: microservices` is selected in Step 1.

---

## 3. Interaction Notes

| Preceding Decision | Impact on Service Boundaries |
|:---|:---|
| Step 5 (DB Strategy) | Guides data isolation. With `db-per-service`, boundaries encompass isolated persistence stores. |
| Step 7 (Communication) | Boundaries interact over network contracts (REST/RabbitMQ) rather than in-memory calls. |

---

## 4. Contract

### Domain Decomposition & Naming
- Services MUST represent a single cohesive Bounded Context aligned with Domain-Driven Design (DDD).
- Service directories and package names MUST be lowercase kebab-case ending with `-service` (e.g. `auth-service`, `catalog-service`, `order-service`).
- Each service MUST have a single business responsibility defined in its `package.json` description field.

### Database Ownership (`own_db` flag)
- If `own_db: true`: The service MUST encapsulate its own data-access layer, entities, migrations, and database connection lifecycle.
- If `own_db: false`: The service acts as an orchestration, proxy, or gateway service and MUST NOT establish direct database connections.

### Anti-Corruption Layer (ACL)
- When consuming entities from an external service, a service MUST map external DTOs into internal domain models.
- External data models MUST NOT be persisted directly or leak into internal business services without translation.

---

## 5. Illustrative Snippet
<!-- illustrative only -->
```typescript
// Anti-corruption adapter translation contract
export class ExternalUserAdapter {
  static toInternalUser(externalDto: ExternalUserResponseDto): UserSummary {
    return {
      userId: externalDto.sub,
      email: externalDto.email_address,
      isActive: externalDto.status === 'ACTIVE'
    };
  }
}
```

---

## 6. Agent Output Checklist
- [ ] Service names match kebab-case naming format `*-service`.
- [ ] Services with `own_db: false` omit database modules and configurations.
- [ ] Inbound external models pass through adapter or DTO transformation layers.

---

## 7. Common Mistakes
- **Anemic God-Services:** Creating a single generic `backend-service` or `api-service` that handles all business operations.
- **Leaking Upstream DTOs:** Storing or using raw third-party DTOs directly within core domain logic.
