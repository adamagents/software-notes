# Microservices

## Overview
Microservices split a product into independently deployable services aligned with business capabilities. They offer faster team autonomy and targeted scaling but only succeed when platform tooling provides consistent runtime standards, observability, and governance. Without that scaffolding, a microservice program devolves into a distributed monolith with fragile contracts.

## Core Challenges
- **Boundary definition**: Poorly sliced services leak data ownership and require cross-service joins.
- **Operational maturity**: Each service needs CI/CD, metrics, tracing, and runbooks; without automation the platform collapses under toil.
- **Cross-service communication**: Excessive synchronous dependencies recreate tight coupling and cascading failures.
- **Testing and rollout**: Version skew between services and schemas makes integration tests and migrations difficult.

## Architecture Building Blocks
- **Platform team**: Provides repo templates, deployment pipelines, secrets management, service mesh, and security policies.
- **Service contracts**: REST, gRPC, or async events defined via schemas with linting, compatibility gating, and changelog tooling.
- **Data ownership**: Each service owns its database; shared read models flow through events or APIs, not direct table access.
- **Observability stack**: Metrics, logs, traces, alerting, and SLO tooling shipped as part of every service skeleton.
- **Governance**: Architectural review boards, automated dependency scanners, and API catalogs keep the landscape understandable.

## Design Checklist
- Conduct domain-driven design workshops to identify bounded contexts before cutting code.
- Enforce single responsibility per service; if a team cannot describe its service capability in one sentence, split further or keep monolithic.
- Prefer asynchronous communication for noncritical paths to avoid long chains of blocking HTTP calls.
- Bake retries, deadlines, and circuit breakers into client libraries or service mesh policies.
- Provide centralized identity, authorization, and configuration services rather than bespoke implementations per team.

## Failure Modes and Mitigations
- **Distributed monolith**: Too many synchronous dependencies. Add caches, queues, or read models to decouple workflows.
- **Snowflake stacks**: Limit languages/runtimes to a supported set; otherwise platform tooling becomes impossible to maintain.
- **Data entanglement**: If multiple services share a database, partition by schema ownership and introduce change-data-capture feeds for shared data needs.
- **Observability gaps**: Mandate tracing headers, structured logging, and RED metrics before services can deploy to production.
- **Organizational drift**: Without regular architecture reviews, APIs diverge and duplication explodes. Maintain service catalogs and automated dependency analysis.

## Migration Strategy
1. Prepare platform fundamentals (CI/CD, logging, tracing, secrets, runtime templates).
2. Identify seams in the monolith (e.g., billing, search) and extract read paths first via strangler patterns.
3. Move data ownership gradually: dual-write through change feeds, validate new stores via shadow traffic, then cut over writes.
4. Establish backward-compatible contracts so the monolith and new services can coexist for multiple releases.
5. Measure velocity, incident frequency, and on-call load to confirm the decomposition is delivering value.

## Interview Prompts
1. How would you enforce data ownership boundaries so one microservice cannot directly access another's tables?
2. Describe your debugging strategy when a request touches six services across regions and returns a 500.
3. What metrics prove that moving from a monolith to microservices improved both delivery speed and reliability?
