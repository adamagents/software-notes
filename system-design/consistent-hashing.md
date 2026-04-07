# Consistent Hashing

Consistent hashing spreads ownership of keys across nodes so that membership changes disturb only a small slice of data. It underpins cache clusters, sharded databases, object storage, and load balancers that must grow and shrink without full rebalancing.

## Summary
- **Minimal Churn**: Adding or removing nodes remaps only adjacent key ranges so caches stay warm.
- **Client-Driven Routing**: Once clients share the ring definition, they can pick targets without centralized schedulers.
- **Heterogeneous Capacity**: Node weights or virtual nodes let beefier machines own more of the hash space.
- **Fault Isolation**: Shard ownership is predictable, making impact analysis and recovery scripts straightforward.

## Mechanics
- Hash both nodes and keys into the same numeric space (ring).
- Assign multiple **virtual nodes** per physical node to smooth distribution and support weights.
- Each key belongs to the first node clockwise from its hash value; replicas can be the subsequent N nodes.
- Rendezvous (highest-random-weight) and Jump Hash are alternative implementations that avoid storing a ring but reach the same behavior.

## Design Levers
| Lever | Options | Guidance |
| --- | --- | --- |
| Hash Function | Murmur, xxHash, SipHash | Favor fast, well-distributed hashes; include partition or rack hints in the seed. |
| Virtual Nodes | Fixed count, weighted count | Increase count for small clusters; weight-heavy nodes so they own proportional ranges. |
| Replica Placement | Simple next-N, rack-aware, region-aware | Mix AZ/region info so replicas do not land on the same failure domain. |
| Membership Source | Gossip, control API, service discovery | Prefer eventually consistent registries with watch/notify semantics so clients converge quickly. |

## Operational Playbook
- **Membership Management**: Publish desired weights via a single source of truth (service discovery, config repo) and secure it; stale clients act like partitions.
- **Rolling Changes**: Simulate ring diffs before production changes to quantify churn, then throttle moves to avoid saturating networks.
- **Bootstrap & Snapshotting**: Persist ring seeds or membership snapshots so clients and servers can restart deterministically.
- **Data Migration**: Stream keys in small batches with progress markers; prioritize hot keys first to minimize cache misses.

## Failure Modes & Mitigations
- **Hot Keys**: Detect skew through per-key metrics and mitigate with salted keys, request coalescing, or dedicated shards.
- **Uneven Distribution**: Use more virtual nodes or switch to bounded-load hashing when variance exceeds SLOs.
- **Partial Views**: Clients with stale membership lists may hammer removed nodes; implement TTLs and backoff around membership fetches.
- **Split Rings**: Validate ring integrity via checksums; mismatch alerts highlight configuration drift before data loss occurs.

## Observability & Tooling
- Track per-node ownership percentage, request rate, error rate, and storage utilization; alert when a node deviates from the mean beyond a threshold.
- Expose ring visualization tooling for operators to inspect range assignments and simulate node loss.
- Emit audit logs for membership changes (who changed weights, when, and why) to speed up incident forensics.

## Interview Drills
1. Walk through how consistent hashing keeps a cache cluster warm during scale-out and scale-in events.
2. Compare ring hashing, rendezvous hashing, and jump hash for a polyglot SDK that needs deterministic routing.
3. Explain how you would add rack or AZ awareness to an existing consistent hashing implementation.
