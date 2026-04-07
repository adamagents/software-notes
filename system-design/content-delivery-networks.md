# Content Delivery Networks (CDNs)

CDNs replicate static and semi-dynamic content to edge POPs worldwide, reducing latency and shielding origins from spikes.

## Goals
- **Latency Reduction**: Serve assets within tens of milliseconds from users worldwide.
- **Origin Protection**: Absorb flash crowds, DDoS, and bot traffic before it reaches core services.
- **Security**: Terminate TLS, enforce WAF rules, and hide origin IPs.
- **Observability**: Provide detailed logging for cache hits/misses and threat intelligence.

## Building Blocks
1. **Edge POPs**: Thousands of geographically distributed servers with SSD caches.
2. **Global Anycast DNS**: Routes clients to nearest healthy POP based on BGP.
3. **Cache Hierarchy**: Regional mid-tier caches reduce cold-miss impact on origins.
4. **Control Plane**: APIs / portals for purge, configuration, TLS cert management.

## Cache Configuration
- **TTL Strategy**: Combine long TTLs with surrogate keys and revalidation headers (ETag, Last-Modified).
- **Stale-While-Revalidate**: Serve slightly stale responses while fetching fresh copy.
- **Stale-On-Error**: Keep prior responses to mask transient origin failures.
- **Compression & Image Optimization**: Brotli/Gzip, AVIF/WebP transforms, HTTP/3, QUIC for transport gains.

## Dynamic Content & Edge Compute
- Use edge functions/workers for header manipulation, bot mitigation, or lightweight personalization.
- Collocate authentication or A/B experiment bucketing at edge to avoid round trips.
- Caution: Keep edge logic stateless; rely on KV stores or durable objects sparingly.

## Security Services
- **WAF**: Rulesets for OWASP Top 10, managed bot defense, geo-blocking.
- **mTLS / Client Certs**: Mutual TLS for B2B APIs proxied through CDN.
- **DDoS Shielding**: Always-on scrubbing centers, rate-based rules, and anomaly detection.
- **Tokenized URLs**: Signed URLs/cookies to prevent hotlinking or unauthorized downloads.

## Monitoring & Operations
- Track cache hit ratio, origin offload, per-POP latency, and error rates.
- Enable real-time logs/analytics streams (Kafka, BigQuery) for debugging.
- Automate purge workflows with idempotent APIs; avoid full-cache flush unless necessary.
- Multi-CDN strategies require consistent configs + health-based traffic steering.

## Interview Drills
1. Design a CDN strategy for personalized content that still benefits from edge caching.
2. Explain how you would roll keys/certificates across thousands of POPs without downtime.
3. Describe how to troubleshoot a cache-miss storm causing origin overload.
