# Content Delivery Networks (CDNs)

CDNs push content closer to users by caching responses at globally distributed points of presence (PoPs), reducing latency and shielding origin infrastructure from spikes.

## What CDNs Provide
- **Static Asset Caching**: HTML, CSS, JS, images with cache-control, ETag validation.
- **Dynamic Content Acceleration**: TCP/TLS termination near users, protocol optimizations (HTTP/3, TLS session reuse).
- **Edge Compute**: Functions/workers to run logic (A/B tests, auth, header rewriting) at the edge.
- **Security**: DDoS mitigation, Web Application Firewall (WAF), bot filtering.
- **Load Shedding**: Shield POP forwarders protect origin capacity.

## Cache Strategy
- Set `Cache-Control`, `Surrogate-Control`, and `Vary` headers intentionally.
- Use **surrogate keys/tags** to invalidate groups of assets atomically.
- Prefer **immutable asset URLs** (content hashes) + long TTL to maximize hit ratio.
- Implement **stale-while-revalidate** to serve cached content while refreshing asynchronously.

## Multi-CDN & Failover
- Use DNS load balancing or traffic steering (based on geography, performance, cost) to split load between providers.
- Continuously measure POP performance (RUM + synthetic) to detect regional degradation.
- Keep origin accessible only from CDN IP ranges; rotate allow lists automatically.

## Observability & Operations
- Monitor hit ratio, origin fetch rate, HTTP status breakdown, and cache fill latency.
- Log request metadata (Edge vs. Origin) for debugging cache misses.
- Automate purge workflows with CI hooks for deployments.
- Test failover by forcing region-specific DNS overrides.

## Edge Compute Considerations
- Constrain runtime (memory, CPU) to avoid impacting tail latency.
- Keep business logic stateless; read config via KV stores or signed cookies.
- Validate and sanitize inputs to avoid running untrusted code path at the edge.

## Interview Prompts
1. How would you decide what to cache at the CDN vs. at the application layer?
2. Describe a strategy for invalidating millions of product pages quickly after a price change.
3. What telemetry would you inspect first if a CDN POP suddenly serves stale data?
