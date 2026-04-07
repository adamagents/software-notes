# CAP Theorem & Practical Trade-offs

CAP formalizes the impossibility of guaranteeing Consistency, Availability, and Partition tolerance simultaneously once a network partition occurs. Modern systems embrace partitions as inevitable and choose behavior deliberately when links fail.

## Core Definitions
- **Consistency (C)**: Every read reflects the latest successful write (linearizability).
- **Availability (A)**: Every request receives a non-error response, regardless of data freshness.
- **Partition Tolerance (P)**: The system continues operating despite dropped or delayed network messages.

## Decision Flow
1. **During Normal Operation**: Systems can deliver both C and A; focus on latency vs. cost (PACELC: Else choose between Latency and Consistency).
2. **When Partitions Occur**:
   - **CP Systems** (Spanner, ZooKeeper, etcd): Sacrifice availability to preserve single-writer ordering. Writes require majority quorum.
   - **AP Systems** (Cassandra, Dynamo, Riak): Accept writes in each partition, reconcile later using timestamps, vector clocks, or CRDT merge functions.
3. **Tunable Consistency**: Dynamo-style stores allow clients to pick `R` and `W` quorums; high `R+W` leans CP, low `R+W` leans AP.

## Design Checklist
- **Business Impact Analysis**: Quantify the cost of stale reads vs. hard errors. Payments usually pick CP; social feeds tolerate AP.
- **Conflict Resolution Strategy**: LWW, domain-specific merge (max, sum), CRDTs, or operational transforms for collaborative editing.
- **Client Experience**: For AP, surface freshness indicators or disable critical actions when uncertainty is high.
- **Operational Instrumentation**: Detect partitions via heartbeat latency, gossip convergence, or high replica divergence.

## Beyond CAP
- **PACELC**: Emphasizes that even without partitions you face latency vs. consistency trade-offs (e.g., Spanner waits for TrueTime to bound uncertainty).
- **Jepsen Testing**: Chaos testing suites that inject partitions to verify real guarantees.
- **CALM Theorem**: Programs that are monotonic (only grow) can achieve eventual consistency without coordination.

## Communication Tips
- Use precise language when describing guarantees to stakeholders (e.g., "read-after-write within region" vs. "global consistency").
- Document SLAs around **RPO** (data loss) and **RTO** (downtime) so on-call engineers know which knob to turn during incidents.
- Educate teams that CAP choices apply per operation or subsystem, not entire companies; you can mix CP and AP services across a product.

## Interview Drills
1. Explain how Cassandra can behave like CP for specific tables by adjusting consistency levels.
2. Describe a customer-facing mitigation when an AP system experiences conflicting updates.
3. Outline how you would validate a vendor's CP claim before adopting it for leader election.
