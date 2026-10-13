# ADR-005: Fail-Fast Startup Configuration Validation

## Status
Accepted

## Context
A major source of production outages in Node.js backends is missing or malformed environment variables that go undetected during deployment and only trigger unhandled runtime exceptions when a specific code path is exercised hours or days later.

## Decision
All applications MUST perform strict schema validation (using Zod or class-validator) as the very first operation of application bootstrap, before initializing database connections or binding HTTP listeners. If any required environment variable is missing, mistyped, or out of range, the process MUST print a diagnostic error and terminate immediately with `process.exit(1)`. Direct access to `process.env` outside the `src/config/` module is strictly prohibited.

## Consequences
- **Positive:** Guarantees that no process runs in a misconfigured state. Misconfigured deployments fail instantly at boot, enabling immediate zero-downtime rollback.
- **Positive:** Developers receive typed autocompletion on configuration objects.
- **Negative:** Requires updating schema definitions whenever new environment variables are added.
