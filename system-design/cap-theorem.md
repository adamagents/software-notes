# CAP Theorem & Trade-offs

CAP states that a distributed data system can provide at most two of Consistency, Availability, and Partition tolerance simultaneously during a network partition. Partition tolerance is non-negotiable in real systems, so the practical decision is between consistency and availability when partitions occur.

## Definitions
- **Consistency (C)**: Every read sees the most recent successful write (linearizability).
- **Availability (A)**: Every request receives a (non-error) response, without guarantee it is the latest data.
- **Partition Tolerance (P)**: The system continues operating despite arbitrary message loss or delay.

## Practical Interpretation
- During normal operation with no partitions, well-designed systems can deliver both C and A.
- During a partition, systems must pick:
  - **CP**: Reject or delay operations that would violate consistency (e.g., majority-quorum databases like etcd, Spanner when writes lose quorum).
  - **AP**: Accept writes in multiple partitions and reconcile later (e.g., Dynamo-style stores, Cassandra with CL=ONE).

## Designing with CAP
1. **Define Business Requirements**: What is the cost of stale data vs. the cost of downtime?
2. **Classify Workloads**:
   - Critical configuration or payment ledgers → favor CP.
   - Social activity feeds, IoT metrics → often favor AP with reconciliation.
3. **Tune Consistency Levels**: Many systems offer tunable consistency (`read quorum + write quorum > replication factor`). Operators can switch between CP-like and AP-like behavior per request.
4. **Plan Conflict Resolution**:
   - Last-write-wins (timestamps, vector clocks)
   - Merge functions / CRDTs
   - Application-specific domain rules (e.g., max balance cannot decrease)

## Beyond CAP
- **PACELC**: If there is a Partition (P) choose between Availability (A) and Consistency (C); Else (when no partition) decide between Latency (L) and Consistency (C). Highlights that low latency and strong consistency also trade off when the network is healthy.
- **Jepsen Testing**: Use fault-injection testing to validate documented guarantees.

## Interview Prompts
1. Explain how Cassandra can be both AP and CP depending on chosen consistency levels.
2. Describe a scenario where you would sacrifice availability for strong consistency.
3. How would you communicate CAP trade-offs to product stakeholders making SLA commitments?
