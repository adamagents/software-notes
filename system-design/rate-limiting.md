# Rate Limiting

Rate limiting protects services from overload, enforces fair use, and underpins monetization tiers.

## Goals
- **Protect infrastructure** from abuse or buggy clients.
- **Guarantee QoS** for premium customers through differentiated limits.
- **Provide graceful degradation** instead of total failure during spikes.

## Algorithms
| Algorithm | Description | Pros | Cons |
| --- | --- | --- | --- |
| Fixed Window | Count requests per window (e.g., per minute) | Simple | Burst at boundary can double traffic |
| Sliding Window Log | Store timestamps for each request, prune outside window | Precise | High memory for large scales |
| Sliding Window Counter | Weighted combination of current + previous window | Smooths bursts | Slight estimation error |
| Token Bucket | Tokens refill at steady rate, bursts allowed up to bucket size | Flexible, handles bursts | Requires sync state |
| Leaky Bucket | Queue requests, drain at constant rate | Smooth output rate | Queues can grow large |

## Architecture Patterns
- **Edge Enforcement**: CDN/WAF rate limits malicious IPs before traffic hits origin.
- **API Gateway Limits**: Apply per API key, user, tenant; integrate with billing.
- **Service Mesh / Sidecar**: Local rate limits for east-west traffic.
- **Distributed Counters**: Use Redis/ DynamoDB with Lua/increments for shared limits; ensure atomicity.

## Implementation Considerations
- Normalize keys (`user:{id}`, `ip:{ip}`, `org:{orgId}`) and support multi-dimensional policies.
- Provide descriptive error responses (`429 Too Many Requests`) with `Retry-After` headers.
- Offer idempotency keys to prevent double-charging when clients retry.
- Instrument allow/deny counts to detect abuse attempts.

## Advanced Topics
- **Adaptive Limits**: Increase thresholds during known marketing campaigns, reduce when downstream dependency is degraded.
- **Shadow Mode**: Evaluate new policies without enforcement to confirm they behave as expected.
- **Hierarchical Limits**: Combine global org limit + per-user + per-endpoint caps.
- **Multi-Region Sync**: Use gossip or CRDT counters to share state across PoPs.

## Interview Prompts
1. How would you design tenant-aware limits for a multi-tenant SaaS API?
2. What happens when your rate limit store (Redis) becomes unavailable?
3. How do you communicate limits and remaining quota to clients?
