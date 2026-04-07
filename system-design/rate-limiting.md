# Rate Limiting

## Overview
Rate limiting protects shared infrastructure, enforces fairness, and contains abuse by capping how fast clients can issue requests. A mature limiter combines accurate measurement, distributed coordination, and transparent feedback so well-behaved clients can self-throttle while attackers are stopped early.

## Core Challenges
- **Distributed accuracy**: Multi-region deployments must keep counters in sync without becoming a single point of failure.
- **Fairness policy**: Limits vary by tenant, endpoint, credential strength, and monetization tier.
- **User experience**: Rejections should include actionable error messages and `Retry-After` metadata.
- **Bypass resistance**: Attackers might rotate IPs, spoof headers, or switch protocols; enforcement must happen at every ingress.

## Architecture Building Blocks
- **Algorithms**: Token bucket (bursty but bounded), leaky bucket (smooth output), fixed window, sliding window counter/log, or probabilistic sketching for very high cardinality.
- **Enforcement points**: API gateway, reverse proxy, service mesh filters, or per-service middleware. Client-side hints improve UX but are never authoritative.
- **State stores**: Redis, Memcached, DynamoDB, or in-memory sharded ledgers hold counters and expiration times.
- **Management plane**: Configuration store, audit logs, UI/CLI for adjusting limits, and alerting when limits are hit abnormally.

## Design Checklist
- Layer limits (IP, user, org, endpoint, authenticated session) so a single compromised key cannot drain all capacity.
- Normalize identities before counting: canonicalize IPs, respect IPv6, and tie requests to auth claims when possible.
- Surface limit headers (`X-RateLimit-Limit`, `-Remaining`, `-Reset`) plus structured error bodies so clients can auto-backoff.
- Include per-endpoint budgets to prevent expensive admin APIs from starving cheaper ones.
- Make configuration dynamic and auditable; on-call engineers must raise limits safely during events.

## Failure Modes and Mitigations
- **Clock skew**: Distributed windows drift. Use centralized stores or hybrid logical clocks, or choose algorithms (token bucket) that rely less on aligned windows.
- **Unlimited internal calls**: Shadow-limit internal service traffic to detect accidental storms even if rejections are suppressed.
- **Shared counters overloaded**: Hot keys in Redis cause contention. Shard counters via consistent hashing or use CRDT-based ledgers.
- **Ineffective blocking**: Attackers switch to WebSockets or gRPC if only HTTP REST paths are protected. Apply policy uniformly across transports.
- **Silent throttling**: If logs omit limit metadata, debugging becomes impossible. Log identity, counter state, and decision per rejection.

## Observability and Operations
- Track limit hits, rejections, datastore latency, error budget impact, and top offenders.
- Alert when total rejections spike, when datastore replication lag grows, or when limit configs change without approval.
- Provide dashboards that correlate rate-limit hits with API versions, regions, and auth scopes.
- Build replay tooling that inspects offender traffic and tests new policies before deployment.
- Run load tests that simulate bursts across regions to validate propagation delay and accuracy.

## Interview Prompts
1. Design a global rate limiter for an API served from three active-active regions with 30 ms replication lag.
2. Explain why token bucket is often preferred over fixed windows for customer-facing APIs.
3. Describe how you would exempt internal services while still metering them for anomaly detection.
