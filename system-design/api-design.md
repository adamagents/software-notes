# API Design Principles

Great APIs are explicit about contracts, predictable in evolution, and defensive against abusive traffic.

## Guiding Tenets
- **Consumer Empathy**: Start from top user journeys; avoid leaking internal schemas.
- **Explicit Contracts**: Machine-readable specs (OpenAPI, gRPC protobufs, GraphQL SDL) describe inputs, outputs, and error envelopes.
- **Backward Compatibility**: Non-breaking additions by default; breaking changes require opt-in versions and migration paths.
- **Defense in Depth**: Auth, rate limits, schema validation, and observability built-in rather than bolted on later.

## Interface Styles
| Style | Strengths | Watch Outs |
| --- | --- | --- |
| REST/HTTP | Cacheable, ubiquitous tooling | Over/under-fetching, version sprawl |
| GraphQL | Flexible queries, typed schema, single endpoint | Resolver N+1, caching complexity |
| gRPC | Compact binary payloads, streaming, strong typing | Browser support limited, steeper learning curve |
| Async (Webhooks, SSE, WebSockets) | Push-based updates, real-time | Retry/dedupe, client reconnection handling |

## Resource Modeling
- Use nouns and hierarchy: `/users/{id}/settings` not `/getUserSettings`.
- Support filtering, sorting, pagination consistently (cursor-based for unbounded lists).
- Separate commands (mutations) from queries; enforce idempotency keys for non-idempotent operations.
- Document field-level semantics, units, and accepted ranges.

## Versioning Strategies
- **URI Versioning** (`/v1/`): Simple routing, but duplicates resources.
- **Header/Content Negotiation**: Cleaner URLs; requires tooling and documentation.
- **GraphQL**: Prefer schema evolution with deprecation warnings rather than explicit versions.
- Publish changelogs, sunset timelines, and adoption metrics before removing fields.

## Error Handling & Observability
- Standard envelope: `{ "error": { "code": "RATE_LIMIT", "message": "...", "details": [] } }`.
- Map domain errors to HTTP semantics (400 validation, 401 auth, 409 conflict, 429 throttling, 5xx server faults).
- Include correlation IDs and request IDs in responses and logs for tracing.
- Emit RED metrics (rate, errors, duration) per endpoint; capture schema version in telemetry.

## Governance & Tooling
- Lint specs for naming conventions, pagination rules, and consistent status codes.
- Use contract tests to validate both server and client implementations.
- Provide SDKs, Postman/Insomnia collections, and runnable examples to reduce integration time.
- Offer sandbox environments with deterministic data sets for partner testing.

## Security Considerations
- Enforce least-privilege scopes; rotate secrets and support OAuth2/OIDC where possible.
- Guard against injection (validate payloads against schema), replay (timestamps + HMAC), and mass assignment attacks.
- Rate limiting, quotas, and behavioral analytics protect shared infrastructure.

## Interview Drills
1. Describe how you would roll out a breaking change that touches thousands of clients.
2. Explain what idempotency means for POST endpoints and how to implement it safely.
3. Compare GraphQL vs. REST for an internal developer platform with diverse clients.
