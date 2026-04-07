# Consistent Hashing

Consistent hashing spreads keys across nodes so that membership changes only remap a small slice of data, making it perfect for caches, sharded databases, and distributed queues.

## Mechanics
1. Hash both nodes and keys into the same numeric space (ring).
2. Each node is replicated into multiple **virtual nodes** to smooth distribution.
3. A key belongs to the first node clockwise from its hash point.
4. Adding/removing nodes only shifts ownership of adjacent segments, minimizing data movement.

## Why It Matters
- **Scalable Client Routing**: Clients can pick destinations without central coordinators once they share the membership list.
- **Heterogeneous Capacity**: Assign more virtual nodes to beefier machines so they own more of the ring.
- **Operational Safety**: Rolling restarts or AZ failures only churn a bounded fraction of keys.

## Design Options
| Variant | Properties | Use When |
| --- | --- | --- |
| Ring Hashing | Visualizable, intuitive | When membership changes slowly |
| Rendezvous (HRW) | No ring structure, O(N) scoring per key | Small clusters or SDKs needing simplicity |
| Jump Hash | O(1) time, deterministic | Very large clusters where memory is tight |
| Bounded Loads | Limits how much any node can exceed average | Multi-tenant caches with hot keys |

## Implementation Tips
- Use high-quality hashes (Murmur3, xxHash) to avoid skew.
- Persist ring snapshots or seeds so client libraries can reproduce the same mapping after restarts.
- Mix rack/zone information into replica selection to avoid co-locating replicas.
- Provide admin tooling to simulate node failure or weight changes before production rollout.

## Handling Edge Cases
- **Hot Keys**: Detect via metrics, split them with salted keys, or pin them to dedicated shards.
- **Membership Convergence**: Rely on gossip/control planes (Consul, ZooKeeper, custom APIs) and treat partial views as partitions.
- **Data Rebalancing**: Stream ownership changes in the background with throttled movers; prioritize high-value keys first.

## Observability
- Track key distribution histograms, per-node ownership percentage, request rate, and error counts.
- Emit audit logs when membership changes to support incident timelines.

## Interview Drills
1. Walk through how consistent hashing keeps cache hit rates higher when scaling up or down.
2. Compare ring hashing and rendezvous hashing for a language-agnostic client library.
3. Explain how you would add rack awareness to a consistent hash implementation.
