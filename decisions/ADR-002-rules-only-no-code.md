# ADR-002: Rules-Only, Contract-Driven Standards (Zero Application Code)

## Status
Accepted

## Context
Traditional starter kits, templates, and boilerplate repositories rot rapidly as runtime dependencies, security patches, and language versions advance. Additionally, AI agents trained on large datasets frequently copy legacy boilerplate patterns verbatim rather than synthesizing idiomatic, up-to-date solutions tailored to modern specifications.

## Decision
This standards repository contains strictly rules, contracts, decision trees, and intake schemas. It contains **ZERO application code**, runnable scripts (`.sh`, `.ts`, `.js`), or Dockerfiles. Code examples are limited to non-runnable illustrative snippets $\le 15$ lines. The AI agent generates all application code dynamically in an external target directory based on these contracts.

## Consequences
- **Positive:** Standards do not suffer from bit rot or dependency vulnerabilities.
- **Positive:** AI agents generate code targeting current stable library releases verified at generation time.
- **Negative:** Requires rigorous conformance checklists and automated verification loops (`build`, `lint`, `test`) to validate generated output.
