# API Design Principles

Robust APIs minimize ambiguity, evolve predictably, and protect downstream systems from abusive or accidental misuse.

## Guiding Tenets
- **Consumer Empathy**: Define APIs around use cases, not internal data models.
- **Explicit Contracts**: Schemas and OpenAPI/GraphQL SDL describe inputs, outputs, and error shapes.
- **Backward Compatibility**: Never break existing clients without version negotiation.
- **Defense in Depth**: Rate limiting, authN/Z, schema validation, and observability built-in from day one.

## Interface Styles
| Style | Strengths | Watch Outs |
| --- | --- | --- |
| REST | Cacheable, simple, HTTP-native | Version sprawl, under/over-fetching |
| GraphQL | Single endpoint, typed schema, flexible queries | Expensive resolver N+1s, complex caching |
| gRPC | Strong types, bidi streaming, compact | Browser support limited, steeper tooling |
| Async APIs (Webhooks, SSE) | Real-time updates | Retry + dedupe complexities |

## Resource Modeling
- Use nouns, not verbs: `/users/{id}/settings`.
- Support filtering, sorting, pagination consistently (cursor-based for large datasets).
- Separate **commands** (side-effect operations) from **queries** for clarity.

## Versioning Strategies
- **URI Versioning** (`/v1/`): Easy to route but invites drift.
- **Header/Content Negotiation**: Cleaner URLs, but clients must set headers correctly.
- **GraphQL**: Prefer schema evolution with deprecations over explicit versions.
- Maintain changelog + sunset policy; instrument actual usage before removing fields.

## Error Handling
- Consistent envelope: `{ "error": { "code": "RATE_LIMIT", "message": "...", "details": [...] } }`
- Map domain errors to HTTP status codes (4xx client issues, 5xx server issues).
- Include correlation/request IDs for debugging.

## Governance
- Automated linting of API specs (naming, pagination, enums).
- Design reviews with consumer teams; create style guides.
- Track adoption metrics and SLA adherence in dashboards.

## Documentation & DX
- Provide runnable examples, SDKs, and Postman/Insomnia collections.
- Document auth flows, retry semantics, idempotency keys, and pagination patterns.
- Offer sandbox environments with deterministic data for integration testing.

## Interview Prompts
1. How would you roll out a breaking change to an API that has thousands of clients?
2. What does "idempotency" mean for POST endpoints, and how do you enforce it?
3. Compare when you would pick GraphQL over REST for an internal platform.
