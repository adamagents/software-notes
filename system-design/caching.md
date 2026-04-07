# Caching Strategies

Caching complements primary data stores by keeping hot data close to consumers, reducing latency and downstream load when applied thoughtfully.

## Layers of Caching
1. **Client / Browser** – HTTP cache-control headers, service workers, offline-first patterns.
2. **CDN / Edge** – Static assets, API responses with surrogate keys, TLS session tickets.
3. **Application Tier** – In-memory caches (LRU, ARC), sidecar memcached/Redis.
4. **Database / Storage** – Buffer pools, page caches, query result caches.

## Cache Design Decisions
- **Key Design**: Use clear namespaces (`user:{id}:profile`) to avoid collisions and enable bulk invalidation via prefix scans.
- **Value Format**: Store encoded structs (JSON, protobuf) with version metadata to support schema evolution.
- **TTL vs. Write-Through**: Short TTLs simplify invalidation but risk thundering herds; write-through keeps cache correct but increases write latency.
- **Consistency Strategy**: Choose between cache-aside (read-through), write-through, write-behind, or read-repair depending on tolerance for stale data.

## Patterns & Anti-Patterns
- **Cache-Aside (Lazy Loading)**: Application checks cache, reads DB if miss, then populates cache. Simple, but prone to stampedes.
- **Read-Through**: Cache provider fetches from origin automatically. Great when using managed caches with hooks.
- **Write-Through**: Writes hit cache and origin simultaneously; ideal for reads that must reflect latest state immediately.
- **Write-Behind**: Batch updates to origin for throughput, but requires durable queues and replay logic.
- **Negative Caching**: Cache "not found" responses briefly to stop repeated misses.
- **Dogpile Protection**: Use request coalescing, mutexes, or probabilistic early expiration (`expire_at - jitter`) to avoid herd effects.

## Capacity Planning
- Measure object size distribution and hit ratios to determine working set.
- Apply **TinyLFU** or **Segregated LRU** when request frequency is highly skewed.
- Track eviction rate vs. miss rate; rising evictions without traffic growth indicates underprovisioning.

## Security & Governance
- Separate namespaces or clusters for PII vs. public data, enforce TLS and AUTH tokens for Redis/memcached.
- Sanitize cacheable responses to prevent user-specific data leaking at higher cache layers.
- Audit cache key entropy to avoid unbounded growth from adversarial inputs.

## Observability
- Instrument per-operation latency, hit/miss counts, evictions, connection pool usage.
- Expose cache effectiveness per endpoint to guide where caching actually helps.

## Interview / Design Prompts
1. How would you design a cache strategy for a social feed with strict freshness for writes but tolerant reads?
2. Describe how to prevent a thundering herd when a popular key expires.
3. When would you bypass caches entirely for a request, and why?
