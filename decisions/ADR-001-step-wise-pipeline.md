# ADR-001: Step-Wise Dependency Decision Pipeline

## Status
Accepted

## Context
When AI agents scaffold backend applications, they frequently suffer from decision paralysis or hallucinate incompatible tech stacks when asked to generate an entire architecture in a single unconstrained prompt (e.g. attempting to pair MongoDB with relational TypeORM migrations or scaffolding microservice monorepos for single-domain applications).

## Decision
We enforce a strictly ordered, 11-step dependency pipeline (Architecture $\rightarrow$ Runtime/Framework $\rightarrow$ Database Engine $\rightarrow$ Data Access $\rightarrow$ DB Strategy $\rightarrow$ Services $\rightarrow$ Inter-Service Comm $\rightarrow$ Shared Code $\rightarrow$ Config $\rightarrow$ Auth $\rightarrow$ Quality). Each subsequent step presents only the options validated by prior choices. Steps specific to microservices are explicitly skipped for monolithic architectures.

## Consequences
- **Positive:** Eliminates incompatible tech stack configurations before generation begins.
- **Positive:** Enables deterministic, selective loading of relevant standard files.
- **Negative:** Requires an interactive interview or a pre-populated `project-spec.yaml` prior to scaffolding.
