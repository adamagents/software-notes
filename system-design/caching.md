# Caching Strategies

Caching narrows the gap between fast consumers and slower systems of record, but only works when invalidation, capacity, and governance are explicit parts of the design.

## Objectives
- **Latency Reduction**: Serve hot data from memory or close to the edge in sub-millisecond time.
- **Load Shedding**: Protect databases downstream by absorbing read bursts.
- **Cost Control**: Shift repeated work to cheaper tiers (edge POPs, hosts with large RAM).
- **Graceful Degradation**: Continue serving mostly-correct results while origins recover.

## Layers of a Cache Stack
1. **Client-Side**: HTTP cache headers, service worker stores, mobile persistent caches.
2. **Edge / CDN**: Surrogate keys, signed URLs, and stale-while-revalidate to protect origins globally.
3. **Application Tier**: Redis/Memcached sidecars, process-local LRU caches for ultra-low latency.
4. **Database Layer**: Buffer pools, query plan caches, and materialized views reduce disk seeks.

## Data Modeling & TTLs
- Namespace keys with clear ownership (`feed:v1:user:{id}`) to enable selective flushes.
- Store schema version + checksum inside cached values to detect incompatible readers.
- TTL Tuning: Short TTL for volatile data, long TTL plus explicit invalidation for mostly static assets.
- Apply jitter to TTLs (`ttl * random(0.9, 1.1)`) to avoid synchronized expirations.

## Read/Write Patterns
| Pattern | Description | Use When |
| --- | --- | --- |
| Cache-Aside | App loads data on miss, writes to cache | Simple adoption, tolerant of stale data |
| Read-Through | Cache provider fetches data | Managed caches, transparent for callers |
| Write-Through | Write to cache and origin atomically | Reads must observe latest state |
| Write-Behind | Buffer writes and flush async | High write throughput, eventual consistency |
| Refresh-Ahead | Background workers refresh popular keys before expiry | Predictable traffic cycles |

## Consistency & Stampede Control
- Use mutexes or request coalescing (single-flight) so only one request regenerates an expired key.
- Negative caching prevents repeated lookups for missing data; pair with very short TTL.
- Idempotent writes + version tokens allow safe retries when caches and databases race.

## Capacity Planning
- Measure object size histogram and hit ratios to size clusters.
- Track eviction reason codes; a spike in evictions with constant traffic signals underprovisioning.
- For sharded caches, monitor ownership imbalance; add consistent hashing with virtual nodes to redistribute smoothly.

## Security & Governance
- Require TLS and AUTH on managed caches; rotate credentials via secrets manager.
- Segment caches by data classification (PII vs. anonymous) to simplify compliance.
- Expire personally identifiable data aggressively and back it with secure erase policies.

## Observability Checklist
- Export hit/miss, latency, memory usage, evictions, replication lag, and connection pool depth.
- Annotate dashboards with deployment/flush events to correlate cache churn with incidents.
- Sample payloads to detect serialization drift or oversize values creeping in.

## Interview Drills
1. Design a cache hierarchy for a personalized feed with strict write freshness but tolerant reads.
2. Explain how you would prevent a thundering herd when a viral key expires.
3. Describe a plan for migrating from cache-aside to write-through without downtime.
