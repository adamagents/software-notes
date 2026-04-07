# Content Delivery Networks (CDNs)

CDNs replicate static and semi-dynamic content to edge points of presence (POPs) around the globe, shielding origins from spikes and keeping latency predictable.

## Summary
- **Latency Compression**: Serve assets within tens of milliseconds by keeping them close to users.
- **Origin Protection**: Absorb flash crowds, bot traffic, and DDoS attacks before they reach core services.
- **Programmable Edge**: Edge logic (workers, functions) allows lightweight personalization and security enforcement near the user.
- **Operational Visibility**: Real-time logs, analytics, and alerting explain cache behavior and security posture.

## Architecture Building Blocks
- **Global Anycast DNS**: Routes clients to the nearest healthy POP using BGP announcements and health checks.
- **POP Hierarchy**: Leaf POPs cache hot objects; mid-tier caches reduce miss storms to the origin.
- **Control Plane**: APIs/portals for purge, configuration, deployment, TLS lifecycle, and log streaming.
- **Edge Compute**: Serverless runtimes for header manipulation, bot mitigation, A/B bucketing, and light personalization.

## Cache Configuration
- **TTL & Validation**: Combine long TTLs with validation headers (ETag, Last-Modified) and surrogate keys for targeted purges.
- **Stale-While-Revalidate / Stale-On-Error**: Serve outdated content briefly while fetching a fresh copy or when origins fail.
- **Variant Keys**: Include headers such as language, device, auth tier, or AB bucket only when necessary to avoid cache fragmentation.
- **Compression & Optimization**: Enable Brotli/Gzip, HTTP/3, and media transforms (WebP, AVIF) to reduce payload size.

## Edge Compute & Personalization
- Keep edge logic stateless; store shared data in globally replicated KV stores sparingly.
- Perform lightweight authentication or experiment bucketing before reaching the origin to save hops.
- Validate third-party scripts and enforce CSP/headers at the edge to prevent injection.
- Monitor cold start times and CPU limits; heavy compute should stay in core services.

## Security Services
- **WAF & Bot Defense**: Managed rules for OWASP Top 10, rate-limiting, bot scoring, and geo-blocking.
- **mTLS & Signed URLs**: Require client certificates or tokenized URLs to protect APIs and private media.
- **DDoS Shielding**: Always-on scrubbing centers absorb volumetric attacks; anomaly detection spots layer-7 floods.
- **Origin Cloaking**: Restrict origins to accept only CDN IP space and rotate secrets frequently.

## Observability & Operations
- Track cache hit ratio, origin offload percentage, request/error rate per POP, and TLS handshake success.
- Stream logs/analytics to centralized stores (Kafka, BigQuery, S3) for debugging and threat hunting.
- Automate purges via idempotent APIs; avoid full-cache clears unless absolutely necessary.
- Multi-CDN strategies require consistent configs plus health-based or performance-based traffic steering.

## Interview Drills
1. Design a CDN strategy for personalized pages that still benefits from caching and edge logic.
2. Explain how you would rotate TLS certificates across thousands of POPs without downtime.
3. Describe how to troubleshoot a sudden drop in cache hit ratio that is overloading the origin.
