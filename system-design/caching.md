# Caching

## Overview
Caching compresses latency and shields origins by serving hot data from faster tiers. A disciplined cache strategy defines ownership, invalidation, observability, and failure posture for every piece of data. Treat caches as subsystems with SLAs instead of opportunistic shortcuts and you gain predictable performance improvements without corrupting correctness.

## Core Challenges
- **Invalidation accuracy**: Keys must encode schema version, tenant, and locale so evictions are surgical.
- **Stampede control**: Surges of misses can DDoS origins unless single-flight or request coalescing is in place.
- **Data sensitivity**: PII and regulated content require encryption, ACLs, and shorter TTLs even when cached.
- **Layer coordination**: Browsers, CDNs, application caches, and database buffers must agree on freshness rules.

## Architecture Building Blocks
- **Client and browser caches**: HTTP caching headers, service workers, and local storage suppress duplicate requests.
- **Edge caches**: CDN POPs cache static assets, API GET payloads, and even generated HTML with stale-while-revalidate policies.
- **Application caches**: Redis or Memcached store computed responses, session state, token buckets, and feature flags.
- **Storage-engine caches**: Buffer pools, page caches, and materialized views keep hot pages on SSD or RAM to reduce random I/O.
- **Control plane**: Namespaces, TTL defaults, invalidation APIs, and audit logs live here to keep caches governable.

## Design Checklist
- Choose cache strategy per dataset: cache-aside for easy retrofits, read-through for managed caches, write-through when strict freshness is required, and refresh-ahead for predictable workloads.
- Namespaces should include dataset version plus optional feature flag or experiment ID (`profile:v3:user:{id}`).
- Apply TTL jitter so expiring entries do not thundering herd the origin at the same instant.
- Cache negative lookups briefly to protect databases from brute-force scans of missing identifiers.
- Document consistency guarantees for every key space (best-effort, bounded-staleness, write-through) so downstream services can reason about correctness.

## Failure Modes and Mitigations
- **Thundering herd**: Use request coalescing, single-flight locks, or asynchronous regeneration so only one caller recomputes an entry.
- **Silent staleness**: Embed schema version or checksums inside cached blobs; delete entries automatically when deployments bump versions.
- **Hot keys**: Detect keys that consume disproportionate CPU and shard them with salts, request coalescing, or dedicated partitions.
- **Replica lag**: Monitor replication delay in distributed caches and prefer local reads with stale-on-failure policies.
- **Cache poisoning**: Authenticate writers, validate payload size and schema, and restrict cross-tenant access.

## Observability and Operations
- Track hit ratio, miss ratio, eviction causes, memory usage, latency, and key size histograms per namespace.
- Emit cache-layer spans in distributed tracing so developers can see when they are serving stale or cold data.
- Provide tooling for targeted invalidation, cache warmups, and snapshot restore after maintenance events.
- Run chaos drills that flush caches or kill nodes to ensure origins survive cold starts.
- Classify cached data by sensitivity and enforce separate clusters or encryption for high-risk categories.

## Interview Prompts
1. Design a caching plan for a personalized feed that requires read-after-write consistency for the posting user but can tolerate stale reads for others.
2. Explain how you would retrofit cache stampede protections into an API that currently calls the database on every miss.
3. Discuss the operational signals you would instrument before launching a new edge cache tier in front of an existing API.
