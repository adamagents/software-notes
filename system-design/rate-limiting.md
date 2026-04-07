# Rate Limiting

Rate limiting protects shared infrastructure, enforces fairness, and constrains abuse by bounding how often clients may call APIs.

## Objectives
- **Prevent Overload**: Keep downstream services within safe QPS and memory limits.
- **Fairness**: Allocate budgets per tenant, IP, user, or API key.
- **Abuse Mitigation**: Detect anomalous bursts, credential stuffing, or scraping.
- **Cost Control**: Stop runaway usage from triggering unexpected bills.

## Algorithms
| Algorithm | Description | Pros | Cons |
| --- | --- | --- | --- |
| Token Bucket | Tokens fill at steady rate; requests consume tokens | Allows bursts, simple implementation | Needs clock sync for distributed tokens |
| Leaky Bucket | Queue drains at fixed rate; drop when full | Smooth output rate | Bursts trimmed aggressively |
| Fixed Window Counter | Count requests per interval | Efficient, simple states | Boundary issues (bursts at edges) |
| Sliding Window Log | Track timestamps, drop when count exceeds limit | Precise, fair | Memory/time heavy |
| Sliding Window Counter | Two fixed windows with weighted average | Balance of precision vs. cost | Slight estimation error |

## Deployment Patterns
- **Client-Side SDKs**: Provide early feedback, but cannot be trusted for enforcement.
- **Edge/API Gateway**: Central enforcement with shared state (Redis, Memcached, global table).
- **Distributed Sidecars**: Service mesh filters enforce local quotas; rely on aggregated metrics for global fairness.
- **On-Device**: Mobile apps throttle user-triggered actions for better UX even before server rejects.

## State Management
- Use fast in-memory stores (Redis with Lua scripts, in-process maps) for counters.
- Shard keys with consistent hashing; maintain replication/backup to survive node loss.
- For global limits, combine per-region quotas with periodic reconciliation (token replenishment via durable queue).

## Policy Design
- Layered limits: per-IP, per-user, per-endpoint, per-organization.
- Exemptions for internal traffic or premium tiers, but enforce quotas elsewhere to avoid unlimited growth.
- Provide headers (`X-RateLimit-Limit`, `-Remaining`, `-Reset`) so clients can self-throttle.
- Add penalty boxes (temporary bans) after repeated violations.

## Observability & Operations
- Monitor limit hits, rejections, latency impact, counter store health.
- Track high-cardinality metrics carefully; rely on sketches or sampled logs for deep dives.
- Provide admin tooling to adjust limits quickly during incidents or launches.

## Interview Drills
1. Design a global rate limiter for an API that spans three regions with active-active traffic.
2. Explain why token bucket is typically preferred over fixed window counters.
3. Describe how you would exempt internal service-to-service calls without opening security holes.
