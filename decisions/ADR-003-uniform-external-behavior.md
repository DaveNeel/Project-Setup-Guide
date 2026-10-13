# ADR-003: Uniform External Behavior Across Frameworks

## Status
Accepted

## Context
In heterogeneous engineering organizations, teams often debate between opinionated full-featured frameworks (NestJS) and minimalist flexible frameworks (Express). However, frontend clients, API gateways, and observability collectors should not need to care which framework powers a backend service.

## Decision
Regardless of whether NestJS or Express is selected in Step 2, the generated backend MUST expose identical external behavior:
- Exact same success envelope: `{ success: true, data, meta? }`.
- Exact same error envelope: `{ success: false, error: { code, message, details }, correlationId }`.
- Exact same single dynamic health endpoint: `GET /health` verifying database connectivity.
- Exact same single-line structured JSON log records with correlation ID tracking.

## Consequences
- **Positive:** Microservices built on different frameworks can coexist seamlessly behind the same API gateway.
- **Positive:** Observability dashboards, alert rules, and client SDKs operate against a single unified contract.
- **Negative:** Express requires dedicated middleware wrappers to achieve parity with NestJS's built-in filters and interceptors.
