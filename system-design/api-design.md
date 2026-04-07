# API Design Principles

Modern API programs treat interfaces as long-lived products. Contracts must be explicit, evolution must be predictable, and every call should be observable, secure, and resilient.

## Summary
- **Consumer empathy first**: Model resources and workflows from the caller's perspective, not from internal schemas.
- **Contracts as code**: Machine-readable specs (OpenAPI, protobufs, GraphQL SDL) double as documentation, validation, and test fixtures.
- **Change discipline**: Prefer additive changes, publish deprecation timelines, and avoid breaking consumers without opt-in migration paths.
- **Defense in depth**: Authentication, authorization, rate limiting, schema validation, and observability belong in the base platform.

## Architectural Building Blocks
- **API Gateway / Edge Tier**: Terminates TLS, authenticates, enforces quotas, rewrites headers, and routes traffic to backend services.
- **Schema & Contract Repository**: Stores specs, enforces lint rules, and gates deployments through contract testing.
- **Documentation & DX Tooling**: Human-friendly docs, SDKs, Postman/Insomnia collections, examples, and sandbox environments accelerate partner onboarding.
- **Lifecycle Automation**: Version promotion pipelines, changelog publishing, and sunset reminders keep the surface area manageable.

## Interface Styles & Trade-offs
| Style | Best For | Watch Outs |
| --- | --- | --- |
| REST/HTTP | Resource-based CRUD with caching, browser compatibility | Over/under-fetching, version sprawl, limited schema rigor |
| GraphQL | Flexible queries across multiple backends | Resolver N+1, caching complexity, strict query cost controls needed |
| gRPC | Low-latency internal services, streaming workloads | Browser gaps, requires strong typing discipline and tooling |
| Async (webhooks, SSE, WebSockets) | Event-driven integrations, push updates | Signature verification, dedupe/retry handling, connection lifecycle management |

## Resource & Schema Design Checklist
- Model with nouns and hierarchy (`/users/{id}/settings`) and keep verbs for actions that mutate (`/users/{id}:suspend`).
- Provide consistent filtering, sorting, pagination (prefer cursor-based for unbounded lists) and document default limits.
- Require idempotency keys for non-idempotent operations and capture them in logs for forensic analysis.
- Document every field's units, acceptable ranges, and nullability; embed schema versions or etags in responses for cache friendliness.

## Evolution & Governance
- Define a single source for API versions (URI prefix, header, or GraphQL schema) and automate compatibility linting before deploys.
- Publish changelogs, migration guides, and sunset timelines; include contact channels for high-risk consumers.
- Use contract tests (consumer-driven or provider-driven) to guarantee compatibility between SDKs and services.
- Track adoption metrics per version to justify deprecations and to alert lagging customers early.

## Reliability, Security & Compliance Controls
- Enforce OAuth2/OIDC scopes or signed tokens; rotate secrets often and support per-environment credentials.
- Validate payloads against schema (JSON Schema, Protobuf) to block injection and type confusion; sanitize logs.
- Layer rate limiting, quotas, behavioral analytics, and anomaly detection near the edge to stop abuse before it hits core services.
- Define standard error envelopes (`{"error":{"code":"RATE_LIMIT","message":"...","details":[]}}`) and map domain errors cleanly to HTTP semantics.

## Observability & Developer Experience
- Emit RED metrics (request rate, errors, duration) per endpoint plus saturation metrics (concurrency, queue depths).
- Attach correlation IDs/request IDs and expose them in response headers to aid tracing across services.
- Collect structured access logs including API key, version, idempotency key, and geo to debug client issues quickly.
- Provide quick-start SDKs/tests that run in CI so consumers detect breaking changes before shipping.

## Interview Drills
1. Outline how you would roll out a backwards-incompatible change while supporting thousands of existing clients.
2. Explain the mechanics of enforcing idempotency for POST endpoints across a multi-region deployment.
3. Compare GraphQL, REST, and gRPC for a platform serving internal teams plus third-party partners, highlighting change-management implications.
