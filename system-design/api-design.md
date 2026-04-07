# API Design

## Overview
APIs are long-lived products. They require deliberate modeling, compatibility guarantees, defenses, and developer experience investments. Treat every API change like a contract negotiation: version it, document it, instrument it, and provide migration paths that respect consumers.

## Principles
- **Consumer perspective first**: Model resources and workflows from the caller's job-to-be-done, not from backend table layouts.
- **Contracts as code**: Maintain OpenAPI, gRPC proto, or GraphQL SDL definitions under version control with linting and compatibility gates.
- **Change discipline**: Prefer additive changes, publish deprecation timelines, and communicate migration plans early.
- **Defense in depth**: Authentication, authorization, schema validation, rate limiting, and observability belong at the edge tier.

## Architecture Building Blocks
- **API gateway**: Terminates TLS, authenticates, enforces quotas, rewrites headers, and routes traffic to backend services.
- **Schema registry**: Stores machine-readable specs, powers contract tests, and keeps SDK generation reproducible.
- **Developer tooling**: Documentation portals, SDKs, Postman/Insomnia collections, sandbox environments, and changelog feeds reduce onboarding time.
- **Lifecycle automation**: Version promotion pipelines, contract testing, release notes, and sunset reminders keep the surface area manageable.

## Design Checklist
- Use intention-revealing resource names (`/users/{id}/settings`) and reserve verbs for actions (`/users/{id}:suspend`).
- Provide consistent filtering, sorting, and pagination. Prefer cursor-based pagination for unbounded lists.
- Require idempotency keys for writes that are not naturally idempotent; persist them for forensic analysis.
- Define structured error envelopes with machine-readable codes plus correlation IDs for tracing.
- Document rate limits, throttling headers, retry guidance, and SLA expectations for every endpoint.

## Interface Styles
- **REST/HTTP**: Best for resource CRUD with caching and browser compatibility. Watch for version sprawl and over/under-fetching.
- **GraphQL**: Offers flexible queries and aggregation. Requires cost controls, caching strategies, and resolver performance tracking.
- **gRPC**: Delivers low-latency, strongly typed interfaces for internal services. Browsers need gRPC-Web or HTTP fallback.
- **Async patterns**: Webhooks, Server-Sent Events, WebSockets, and event streams push updates to clients; require signature verification and replay tooling.

## Reliability and Security
- Enforce OAuth2/OIDC scopes or signed tokens per tenant; rotate secrets frequently.
- Validate every payload against schema before touching business logic.
- Layer rate limiting, quotas, anomaly detection, and abuse heuristics near the edge.
- Emit RED metrics per endpoint alongside saturation signals like queue depth and concurrency.
- Log structured access events including API key, version, idempotency key, and geo to aid forensic investigations.

## Evolution and Governance
- Maintain a single source for API versions (URI prefix, header, or schema tag) and enforce compatibility via CI.
- Track adoption metrics per version; use them to justify and schedule deprecations.
- Provide migration guides, changelog entries, and proactive customer communication whenever a breaking change looms.
- Use contract tests (consumer-driven or provider-driven) to keep SDKs and services in sync.
- Store API policies as code so reviews, rollbacks, and audits are straightforward.

## Interview Prompts
1. Describe how you would roll out a backwards-incompatible change to a high-traffic public API without breaking clients.
2. Explain the mechanics of enforcing idempotency for POST requests across a multi-region deployment.
3. Compare REST, GraphQL, and gRPC for a platform serving both internal teams and third-party partners.
