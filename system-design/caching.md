# Caching Strategies

Caching narrows the gap between impatient users and slower systems of record. A successful strategy treats caching as a full subsystem, complete with governance, observability, and failure planning, rather than as a sprinkle of in-memory lookups.

## Summary
- **Latency & Cost**: Serve hot data from memory or the edge within sub-millisecond time while protecting expensive databases and APIs.
- **Predictable Invalidation**: Keys, namespaces, and TTLs must reflect data ownership to keep caches trustworthy.
- **Surge Absorption**: Stampede control and jittered TTLs keep viral events from toppling origins.
- **Data Stewardship**: Security classification, auditing, and privacy policies apply to cached bytes as much as to primary stores.

## Layered Cache Stack
| Layer | Examples | Responsibilities |
| --- | --- | --- |
| Client | HTTP caching headers, service workers, local storage | Offline UX, suppress duplicate network hops |
| Edge/CDN | CDN object caches, surrogate keys, stale-while-revalidate | Global fan-out, origin shielding, TLS termination |
| Application | Redis/Memcached, process-local LRU | Low-latency reads, write buffering, rate limiting tokens |
| Storage Engine | DB buffer pools, materialized views | Reduce disk seeks, serve hot partitions |

## Key Design Levers
- **Key Namespacing**: Encode schema version, locale, and tenant in keys (`feed:v2:user:{id}`) so flushes are surgical.
- **TTL Strategy**: Mix deterministic TTLs for volatile data with long TTL + explicit invalidation for mostly-static assets; always add random jitter to prevent synchronized expirations.
- **Stampede Prevention**: Use single-flight locks, request coalescing, or async refresh-ahead workers so only one miss regenerates data.
- **Negative Caching**: Cache not-found results briefly to avoid repeated origin hits for nonexistent entities.
- **Consistency Contracts**: Document whether readers see eventual or strong consistency; use version tokens or etags to guard upserts.

## Common Read/Write Patterns
| Pattern | Mechanics | Ideal When |
| --- | --- | --- |
| Cache-Aside | App fetches on miss, writes result | Quick retrofit, tolerant of stale data |
| Read-Through | Cache service fetches from origin | Managed caches, drop-in adoption |
| Write-Through | Writes update origin + cache synchronously | Readers require latest state immediately |
| Write-Behind | Buffer and flush asynchronously | Heavy write bursts, eventual consistency acceptable |
| Refresh-Ahead | Background job refreshes hot keys | Predictable access cycles, heavy reads |

## Operational Playbook
- **Capacity Planning**: Track object size histograms, hit/miss ratios, and eviction causes; shard via consistent hashing plus virtual nodes when clusters grow.
- **Data Governance**: Segment PII, financial, or regulated data into dedicated caches with tighter ACLs and TTLs; encrypt at rest where supported.
- **Cold Start Procedures**: Keep warm-up scripts or snapshot restores ready to repopulate caches after maintenance or disasters.
- **Deployment Hygiene**: Tie cache flushes to deploy stages; prefer targeted key invalidation over global purges.

## Failure Modes & Mitigations
- **Thundering Herd**: Mitigate with jittered TTLs, token buckets on regenerate paths, and prewarming high-value keys.
- **Silent Staleness**: Embed schema versions and checksums inside cached blobs; drop entries when readers upgrade.
- **Hot Keys**: Detect via per-key counters, then shard with salts or pin to dedicated shards with weighted hashing.
- **Replication Lag**: Monitor replica lag for clustered caches and degrade gracefully (read local cache, serve stale) when failover occurs.

## Observability & Capacity Signals
- Export hit/miss ratios, latency per cache layer, eviction counts, memory usage, replication delay, and connection pool depth.
- Correlate cache flush commands and deployments on dashboards; most incidents occur immediately after invalidation events.
- Sample payloads to catch serialization drift, oversize values, or PII leakage; alert when values exceed planned size budgets.

## Interview Drills
1. Design a multi-layer cache hierarchy for a personalized feed that requires immediate write visibility but can tolerate slightly stale reads elsewhere.
2. Explain how you would keep a Redis cluster healthy during a viral event when numerous hot keys expire simultaneously.
3. Describe the migration steps from cache-aside to write-through without losing correctness or overloading the primary database.
