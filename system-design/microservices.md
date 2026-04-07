# Microservices Architecture

Microservices decompose a product into independently deployable services with clear domain boundaries. They enable parallel development but demand strong platform rigor.

## Summary
- **Team Autonomy**: Separate deploy cadences for bounded contexts accelerate delivery when platform tooling is mature.
- **Strong Ownership**: Each service owns its data store and contracts; no reaching into another service's tables.
- **Platform First**: CI/CD, observability, security, and runtime templates must exist before large-scale decomposition.
- **Failure Containment**: Circuit breakers, backpressure, and async communication keep faults localized.

## Readiness Checklist
- Domains mapped via DDD/event storming with explicit bounded contexts and shared vocabulary.
- Centralized platform team providing repo templates, deployment pipelines, secrets management, and runtime standards.
- Observability stack (metrics, logs, tracing) with propagation libraries shipped in every service.
- Governance process for schema changes, API versioning, and dependency approvals.

## Design Principles
- **Single Responsibility**: Services align with business capabilities and expose APIs/events that express that capability clearly.
- **Loose Coupling**: Communicate via versioned contracts, avoid synchronous dependency chains when eventual consistency suffices.
- **Automation Over Heroics**: Infrastructure as code, automated rollouts/rollbacks, and policy-as-code to enforce compliance.
- **Data Ownership**: Each service writes to its own store, shares read models via events or APIs, and avoids shared mutable databases.

## Communication Patterns
| Pattern | Use When | Considerations |
| --- | --- | --- |
| REST/gRPC | Low-latency request/response workflows | Add retries, deadlines, circuit breakers |
| Async Events | Decoupled workflows, audit trails, read-model fan-out | Requires idempotent consumers and DLQs |
| Service Mesh | Uniform networking (mTLS, retries) across heterogeneous runtimes | Operational overhead, sidecar resource cost |

## Operational Foundations
- **Observability**: Propagate trace IDs, emit RED/USE metrics per endpoint, and centralize structured logging.
- **Deployments**: Blue/green, canary, and feature-flag rollouts coupled with automated rollback triggers based on SLOs.
- **Configuration Management**: Versioned config store plus runtime overrides with audit trails; secrets rotated centrally.
- **Security**: Zero-trust networking, mutual TLS, fine-grained IAM, and dependency scanning built into CI.

## Anti-Patterns to Avoid
- **Distributed Monolith**: Too many synchronous dependencies recreate monolith coupling. Introduce async boundaries or caches to decouple.
- **Snowflake Services**: Enforce platform-approved languages/frameworks to prevent bespoke stacks per team.
- **Data Entanglement**: Without canonical events, data drifts across services. Establish event backbones and ownership rules.
- **Premature Slicing**: Split only when metrics show a single service as a bottleneck; microservices add overhead.

## Migration Strategy
1. Map current capabilities and identify seams that can be strangled one at a time.
2. Build shared platform components (logging, auth, deployment) before large migrations.
3. Proxy specific routes to new services while the monolith remains the source of truth; move data ownership last.
4. Introduce CDC or change feeds to keep read models synchronized during the transition.

## Interview Drills
1. How would you enforce data ownership boundaries between microservices and prevent unauthorized joins?
2. Describe how to debug a customer request that traverses six services across regions.
3. What metrics prove that decomposing into microservices actually improved delivery velocity and reliability?
