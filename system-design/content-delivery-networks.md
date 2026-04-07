# Content Delivery Networks

## Overview
CDNs replicate content and lightweight logic to edge points of presence so users receive responses from geographically close servers. They also shield origins from volumetric attacks, enforce security policy, and provide observability into client behavior. Treat the CDN as a programmable platform with rigorous configuration management instead of a static cache if you want predictable performance and resilience.

## Core Challenges
- **Cache correctness**: TTLs, validation headers, and surrogate keys must reflect ownership boundaries or the edge will serve stale or private data.
- **Personalization vs. cacheability**: Highly dynamic pages need careful use of edge workers, signed cookies, or hole punching to stay cachable.
- **Operational drift**: Hundreds of POPs magnify config mistakes. Every header change or TLS update must propagate safely.
- **Visibility**: Operators need real-time logs and metrics from the edge to debug incidents before users notice regressions.

## Architecture Building Blocks
- **DNS and anycast routing**: Global load balancing directs users to the nearest healthy POP while preserving ability to shed load from unhealthy ones.
- **Edge cache tiers**: Leaf POPs cache frequently accessed objects; mid-tier or shield POPs reduce miss storms toward the origin.
- **Programmable edge runtime**: Functions or workers modify headers, run bot mitigation logic, or perform light personalization without origin hops.
- **Security services**: WAF, bot detection, DDoS scrubbing, TLS termination, mTLS, and signed URL enforcement live closest to the user.
- **Control plane**: APIs or GitOps-managed configs govern cache keys, TLS certs, purge requests, and log streaming destinations.

## Design Checklist
- Decide caching strategy per asset type: immutable fingerprints (long TTL, no revalidation), semi-dynamic pages (stale-while-revalidate), and sensitive APIs (short TTL plus validation headers).
- Use surrogate keys or tag-based purges so releases flush only the affected objects rather than entire domains.
- Normalize request headers before cache lookup to avoid unbounded key explosion; vary only on attributes that matter (language, auth tier, device class).
- Provide signed URLs or tokens for private media and enforce origin access control so bypass attacks fail.
- Automate TLS certificate issuance, rotation, and validation across every POP.

## Failure Modes and Mitigations
- **Cache poisoning**: Validate origin responses, enforce content security policies, and restrict which headers participate in the cache key.
- **Low hit ratio**: Inspect cache status codes, object size histograms, and TTLs; adjust key normalization or add shield POPs to improve reuse.
- **Origin overload**: Enable circuit breakers and surge queues so mid-tier caches absorb storms even when leaf POPs miss.
- **POP isolation**: Run synthetic probes from every region, and maintain runbooks for rerouting traffic when a POP misbehaves.
- **Edge compute bugs**: Deploy edge worker code gradually with feature flags and automatic rollback when error rates spike.

## Observability and Operations
- Stream request logs, WAF events, and cache metrics to centralized storage for analysis. Redact PII before exporting.
- Track cache hit ratio, origin offload, TLS success rates, HTTP protocol mix, and latency percentiles per POP.
- Integrate CDN metrics with incident response tooling so on-call engineers can correlate cache purge events with user-facing symptoms.
- Use policy-as-code for CDN configs to enable peer review, linting, and automated testing before rollout.
- Rehearse full-site purges and POP failovers so the team understands blast radius and recovery mechanics.

## Interview Prompts
1. Design a CDN strategy for an e-commerce storefront with personalized recommendations but cachable assets.
2. Explain how you would debug a sudden drop in cache hit ratio that coincides with a new marketing campaign.
3. Describe how to rotate customer-managed TLS certificates across thousands of CDN endpoints without downtime.
