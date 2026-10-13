# ADR-004: Strict Repository Layer Boundary

## Status
Accepted

## Context
Codebases that allow controllers and domain services to import ORM query builders, entities, or database driver connections directly become tightly coupled to the data-access technology. Swapping an ORM, testing business logic in isolation, or changing database engines requires rewriting controllers and domain services.

## Decision
All persistence operations MUST sit behind an explicit repository interface (`I*Repository`). Domain services depend exclusively on the interface. Concrete repository implementations encapsulate all ORM/ODM-specific operations (TypeORM, Mongoose, Prisma, Drizzle). Controllers MUST NOT import repositories or ORM entities directly.

## Consequences
- **Positive:** Domain business logic is completely unit-testable in memory using mock repository interfaces without spinning up database containers.
- **Positive:** Data-access libraries can be replaced or upgraded without touching core business services.
- **Negative:** Introduces an interface and mapper class per aggregate, slightly increasing file count.
