# Microservices Architecture

Microservices decompose a product into independently deployable services with clear domain boundaries. They enable parallel development but demand strong platform rigor.

## When Microservices Make Sense
- Organization has multiple teams that need autonomy in deploy cadence.
- Domains are well-understood (bounded contexts) with minimal cross-write coupling.
- Tooling exists for CI/CD, observability, and infrastructure automation.
- Failure blast radius must be minimized.

## Design Principles
- **Single Responsibility**: Each service owns a bounded context plus its data store.
- **Loose Coupling, High Cohesion**: Interactions happen via well-defined APIs or events; avoid reaching into another service's database.
- **Contracts First**: Specs, schema registries, and backward compatibility guardrails in place before implementation.
- **Automation**: Templates for repos, build pipelines, and runtime configs ensure consistency.

## Communication Patterns
| Pattern | Use When | Considerations |
| --- | --- | --- |
| REST/gRPC | Request/response workflows, low latency | Add retries, deadlines, circuit breakers |
| Async Events | Process decoupled workflows, audit trails | Need idempotent consumers & DLQs |
| Service Mesh | Uniform networking, mTLS, retries | Operational overhead, sidecar cost |

## Operational Foundations
- **Observability**: Distributed tracing (propagate trace IDs), structured logs, metrics per service (RED + USE).
- **Deployment**: Blue/green, canary, and feature flags to test gradually; automated rollbacks when SLOs fail.
- **Configuration**: Central config service with versioning; runtime overrides via dynamic config while auditing changes.
- **Security**: Zero-trust networking, mutual TLS, least-privilege IAM, dependency scanning baked into CI.

## Avoiding Pitfalls
- **Distributed Monolith**: Too many synchronous dependencies; introduce async boundaries and caching to decouple.
- **Snowflake Services**: Enforce platform-approved frameworks/libraries to avoid bespoke stacks per team.
- **Data Entanglement**: Without canonical events, data drifts; establish event backbones and ownership rules.
- **Over-Sharding**: Resist splitting until metrics show a single service as bottleneck.

## Migration Strategy
1. Map domains via event storming or DDD.
2. Build shared platform components (logging, auth, deployment) first.
3. Strangle the monolith: proxy specific routes to new services, keep single data source initially.
4. Gradually extract data ownership, adding CDC or change feeds for read models.

## Interview Drills
1. How would you enforce data ownership boundaries between microservices?
2. Describe how to debug a customer request that traverses six services.
3. What metrics prove that decomposing into microservices actually improved delivery?
