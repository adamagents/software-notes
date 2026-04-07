# CAP Theorem

## Overview
The CAP theorem states that in the presence of a network partition, a distributed system can offer either strong consistency or high availability, but not both simultaneously. Modern architectures assume partitions are inevitable and explicitly document which subsystems choose CP behavior versus AP behavior. Mature teams treat CAP (and its sibling PACELC) as guidance for designing user-facing behavior during steady state and failure state.

## Core Concepts
- **Consistency**: Every read receives the most recent write result (linearizability).
- **Availability**: Every request receives a response, even if the response serves stale or approximate data.
- **Partition tolerance**: The system continues operating when messages are lost, delayed, or reordered between nodes.
- **PACELC**: Even when no partition exists, systems trade latency (L) for consistency (C). Spanner waits for clock uncertainty, while Dynamo prioritizes fast responses and reconciles conflicts later.

## Design Approaches
- **CP systems**: Spanner, etcd, Zookeeper, and CockroachDB rely on quorum writes; minority partitions refuse requests until they rejoin.
- **AP systems**: Cassandra, DynamoDB, and Riak accept divergent writes locally and rely on read-repair, hinted handoff, or CRDT merges.
- **Tunable consistency**: Clients pick read (`R`) and write (`W`) quorum sizes relative to replica count (`N`) to tilt between CP and AP semantics on a per-query basis.
- **Hybrid control/data planes**: Keep metadata stores (leader election, feature flags) strongly consistent, while user-facing feeds or analytics run on AP stores.

## Decision Checklist
- Capture business impact of stale data or downtime before picking CP or AP. Payments usually demand CP, while social feeds prefer AP.
- Document how clients experience partitions: read-only mode, disabled writes, or explicit outage messaging.
- Define conflict resolution ahead of time. Strategies include last-write-wins with vector clocks, CRDTs, or domain-specific merge rules.
- Make operations idempotent so retries during partitions do not produce double side effects.
- Provide observability around partition events: heartbeat latency, quorum failure counts, divergence metrics, and automated alerts.

## Failure Modes and Mitigations
- **Hidden partitions**: Gossip health checks might look green while routers block traffic. Trace heartbeats through every hop and include application-level probes.
- **Clock skew**: Timestamp-based merges (LWW) break when clocks drift. Use hybrid logical clocks or incorporate causality metadata.
- **Conflict storms**: Long-lived partitions accumulate divergent updates. Provide tooling for manual reconciliation and user-facing resolution flows.
- **Overly strict fail-closed**: Rejecting all writes for minor blips can degrade UX more than serving slightly stale data. Consider per-endpoint policies.
- **Unbounded staleness**: At-least-once replication without compaction can produce arbitrarily old data. Track max-lag dashboards and enforce SLAs.

## Interview Prompts
1. Walk through designing a shopping cart service where availability is critical but price changes must remain consistent. Where do you draw CP vs. AP lines?
2. Explain how quorum-based reads and writes in Cassandra can be combined to emulate CP behavior for a single keyspace.
3. Outline the testing you would run (Jepsen style) before trusting a third-party coordination service that claims to be CP.
