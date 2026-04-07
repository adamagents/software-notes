# Microservices Architecture

Microservices decompose a system into independently deployable services with clear boundaries, enabling teams to ship faster while managing complexity.

## When to Use
- Organization has multiple teams needing independent deploy cadence.
- Domain boundaries are well-understood (bounded contexts).
- Platform investment (observability, CI/CD, service mesh) exists to support them.
- Scaling requirements differ between components.

## Design Principles
- **Single Responsibility**: One service owns its data + logic for a bounded context.
- **Own Your Data**: No shared database schemas; communicate via APIs/events.
- **Contracts First**: Versioned APIs, schema registries, backward compatibility.
- **Automation Everything**: Templates for repos, CI pipelines, IaC modules.

## Communication Patterns
- **Synchronous**: REST/gRPC for request/response workflows; add retries, deadlines, circuit breakers.
- **Asynchronous**: Events and queues for loosely coupled workflows; enables temporal decoupling.
- **Service Mesh**: Sidecars provide mTLS, retries, rate limits, telemetry uniformly.

## Operational Concerns
- **Observability**: Centralized logging, tracing (Trace IDs propagated end-to-end), RED metrics per service.
- **Deployment**: Blue/green, canary, feature flags; per-service pipelines with automated rollbacks.
- **Configuration**: Consistent secrets management, dynamic config (e.g., etcd/Consul) with safe rollout.
- **Security**: Zero-trust networking, least-privilege IAM, dependency scanning.

## Common Pitfalls
- **Distributed Monolith**: Services tightly coupled via synchronous chains, cascading failures.
- **Snowflake Services**: Each team uses different frameworks; avoid via platform standards.
- **Chatty Communication**: Over-decomposition causing high latency; aggregate where necessary.
- **Inconsistent Data Models**: Without canonical events, data drifts between services.

## Migration Strategy
1. Identify seams in the monolith via domain-driven design.
2. Strangle the monolith: route specific functionality to new service via proxy/gateway.
3. Establish shared platform components before mass extraction.
4. Measure success with deployment frequency, MTTR, and incident counts.

## Interview Prompts
1. How would you enforce data ownership boundaries between microservices?
2. Describe your approach to debugging a request that spans six services.
3. What metrics show that splitting into microservices was successful?
