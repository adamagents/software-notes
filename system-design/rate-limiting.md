# Rate Limiting

Rate limiting protects shared infrastructure, enforces fairness, and constrains abuse by bounding how often clients call APIs or services.

## Summary
- **Prevent Overload**: Keep downstream services within safe QPS and memory limits.
- **Fairness**: Allocate budgets per tenant, IP, user, or API key.
- **Abuse Mitigation**: Detect anomalous bursts, credential stuffing, or scraping.
- **Cost Control**: Stop runaway usage from triggering unexpected bills.

## Core Algorithms
| Algorithm | Description | Pros | Cons |
| --- | --- | --- | --- |
| Token Bucket | Tokens accumulate at steady rate; requests consume tokens | Allows bursts, simple distributed impl | Needs clock sync for distributed ledgers |
| Leaky Bucket | Queue drains at fixed rate; drop when full | Smooth output rate | Clips bursts aggressively |
| Fixed Window Counter | Count requests per interval | Efficient state | Edge-aligned bursts exploit boundary |
| Sliding Window Log | Track timestamps; drop when count exceeds limit | Precise fairness | Higher memory/CPU cost |
| Sliding Window Counter | Weighted sum of adjacent windows | Good balance of precision vs. cost | Slight estimation error |

## Deployment Patterns
- **Client-Side SDKs**: Provide early feedback but cannot be trusted for enforcement.
- **Edge/API Gateway**: Central enforcement with shared state (Redis, Memcached, DynamoDB) for per-tenant quotas.
- **Distributed Sidecars**: Service mesh filters enforce local quotas while aggregating metrics globally.
- **On-Device Throttling**: Mobile or desktop apps self-regulate to avoid poor UX even before server rejection.

## Policy Design
- Layered limits: per-IP, per-user, per-endpoint, per-organization.
- Provide headers (`X-RateLimit-Limit`, `-Remaining`, `-Reset`) so clients can self-throttle.
- Support burst credits and penalty boxes (temporary bans) for chronic offenders.
- Make configuration dynamic with audit logs; incident responders must raise limits safely during launches.

## Failure Modes & Mitigations
- **Clock Skew**: Use centralized stores or hybrid logical clocks so distributed limiters stay aligned.
- **Inconsistent State**: Shard counters via consistent hashing and replicate to reduce single points of failure.
- **Unlimited Internal Traffic**: Define explicit exemptions but continue to meter internal calls to detect runaway services.
- **Bypass Vectors**: Enforce limits at every ingress path (REST, gRPC, WebSocket) and ensure auth tokens cannot be spoofed.

## Observability & Tooling
- Monitor limit hits, rejections, latency impact, and datastore health (connection pool depth, replication lag).
- Track high-cardinality metrics carefully; rely on sketches or sampled logs for deep dives.
- Provide admin tooling to adjust limits quickly, replay offender traffic, and correlate decisions with audit logs.

## Interview Drills
1. Design a global rate limiter for an API that spans three active-active regions.
2. Explain why token bucket is typically preferred over fixed window counters.
3. Describe how you would exempt internal service-to-service calls without opening security holes.
