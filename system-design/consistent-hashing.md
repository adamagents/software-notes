# Consistent Hashing

Consistent hashing distributes keys across nodes so that when nodes join or leave, only a small fraction of keys remap. It underpins scalable caches, sharded databases, and distributed queues.

## Core Idea
- Map the hash space (0..2^m-1) onto a ring.
- Hash each node to multiple points (virtual nodes) on the ring.
- Hash each key; walk clockwise to the first node hash—this node owns the key.
- When nodes change, only the keys in adjacent segments move.

## Advantages
- Minimal rebalancing vs. modulo `hash(key) % N` which reshuffles most keys when N changes.
- Supports heterogeneous capacity by assigning more virtual nodes to larger instances.
- Works with client-side routing; no central coordinator is needed once membership is known.

## Design Considerations
- **Virtual Nodes**: Increase to smooth distribution; 100–200 per node is common.
- **Membership Tracking**: Requires consistent view (ZooKeeper, gossip protocols, control plane APIs).
- **Replication**: Assign successive nodes on the ring as replicas; mix rack/zone awareness to avoid colocating replicas.
- **Hot Keys**: Detect and split heavy keys (e.g., add key-specific salt, move to dedicated shard).
- **Data Movement Cost**: Plan for background rebalancing bandwidth when nodes scale.

## Variants
- **Rendezvous (HRW) Hashing**: Compute `score = hash(node, key)` per node; pick node with highest score. Simpler implementation, naturally supports weighting.
- **Consistent Hashing with Bounded Loads**: Adjust selection to cap maximum load (e.g., `power of two choices` on top of consistent hashing ring).
- **Jump Hash**: O(1) time, O(1) memory hashing for large cluster IDs, great for client SDKs.

## Practical Tips
- Use cryptographic or high-quality non-crypto hash (xxHash, Murmur) to avoid skew.
- Persist metadata (ring snapshots) to recover quickly after control plane restarts.
- Provide tooling to visualize key distribution and simulate node failure before rolling changes.

## Interview Prompts
1. Walk through what happens to cache keys when a node fails in a consistent hash ring.
2. How would you add rack awareness to replica placement on the ring?
3. Compare consistent hashing with rendezvous hashing for a client library.
