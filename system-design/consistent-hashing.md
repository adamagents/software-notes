# Consistent Hashing

## Overview
Consistent hashing maps keys and nodes into the same hash space so cluster membership changes affect only a small slice of traffic. It is the backbone of distributed caches, sharded databases, and peer-to-peer storage engines that must scale elastically without blowing away hot data. Good implementations pair deterministic routing with operational tooling that makes membership changes observable and reversible.

## Core Challenges
- **Smooth distribution**: Limited node counts often produce uneven key ownership unless virtual nodes or bounded-load hashing are applied.
- **Membership propagation**: Clients, gateways, and servers must update their ring definitions quickly or they will disagree about ownership.
- **Failure isolation**: When a node disappears, replicas must be consulted deterministically without creating write hotspots.
- **Capacity weighting**: Clusters often contain heterogenous hardware; the hash scheme must honor weights while keeping churn minimal.

## Architecture Building Blocks
- **Hash function**: Non-cryptographic but well-distributed hashes (Murmur, xxHash) map nodes and keys onto the ring.
- **Virtual nodes**: Multiple positions per physical node smooth randomness and implement capacity weights by allocating proportional vnodes.
- **Replica policy**: Additional clockwise nodes or rack-aware placement ensure redundancy across failure domains.
- **Membership service**: Config repos, gossip protocols, or service discovery APIs publish ring snapshots plus change history.
- **Client libraries**: SDKs or proxies compute placements locally to avoid centralized coordinators.

## Design Checklist
- Encode region, AZ, or rack metadata with every node entry so placement can avoid correlated failures.
- Precompute and version ring snapshots. Store them in durable config stores and embed checksums so clients detect drift.
- Simulate data movement before adding or removing nodes; throttle migrations and prioritize hot keys to keep hit ratios high.
- Pair consistent hashing with load-aware admission control (token buckets, concurrency caps) to defend against hot partitions even with perfect distribution.
- Provide administrative tooling for manual key pinning or emergency overrides when a tenant needs isolation.

## Failure Modes and Mitigations
- **Partial views**: If clients miss an update, they may hammer removed nodes. Use TTLs on membership data and require periodic refreshes.
- **Hot keys**: Some datasets naturally skew. Add per-key rate limiting, request coalescing, or hash-salting to spread demand.
- **Ring splits**: Configuration merges gone wrong can create divergent rings. Validate ring checksums at startup and refuse traffic on mismatch.
- **Replica collisions**: Without rack awareness, consecutive replicas may land in the same AZ. Annotate nodes with topology metadata and select replicas that span faults.
- **Rebalance storms**: Adding many nodes at once moves huge data volumes. Batch the rollout and pause between steps to let caches converge.

## Observability and Operations
- Track per-node ownership percentage, request rate, error rate, and storage usage; alert when deviations exceed thresholds.
- Visualize ring layouts and churn rate so operators understand the impact of membership proposals before approving them.
- Version-control membership definitions, review them via pull requests, and maintain audit logs of who changed what.
- Provide automated rebalancing jobs that stream keys and record progress plus checkpointing for resume after failure.
- Integrate placement decisions with tracing by tagging spans with shard identifiers; debugging becomes faster when logs state which node served each request.

## Interview Prompts
1. Describe how you would add weighted virtual nodes to an existing consistent hashing implementation without causing mass churn.
2. Explain how rendezvous hashing compares to ring hashing for polyglot clients that need deterministic routing without sharing a ring file.
3. Design a troubleshooting plan for a cache cluster experiencing repeated hot key incidents even though hashing is uniform on paper.
