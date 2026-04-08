# Chapter 4: Design a Rate Limiter

## Summary

This chapter designs a distributed server-side rate limiter that is accurate, low-latency, memory-conscious, and fault-tolerant. The practical shape of the system is a rate-limiting service or middleware backed by a fast shared store such as Redis.

## Key Ideas

- Place enforcement on the server side or in gateway middleware, not in the client.
- Support flexible rules keyed by dimensions like IP address, user ID, or endpoint.
- Common algorithms include token bucket, leaky bucket, fixed window, sliding log, and sliding counter.
- Store counters in an in-memory system so checks stay fast under high request volume.
- Return clear `429 Too Many Requests` responses and expose retry information when possible.
- Plan for distributed coordination, clock issues, stale config, and partial store failures.
- Decouple policy storage, configuration rollout, metrics, and enforcement logic.

## Design Shape

- Rules live in a configuration store.
- Request counters live in Redis or another atomic, low-latency data store.
- Middleware consults the limiter before forwarding to application servers.
- Violations are rejected immediately and logged for visibility.

## Interview Takeaway

The algorithm choice matters less than whether you can explain the trade-offs among accuracy, burst handling, memory usage, and operational simplicity.
