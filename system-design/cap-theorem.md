# CAP Theorem & Practical Trade-offs

CAP says you cannot simultaneously guarantee strong Consistency and Availability once a Partition occurs. Mature systems assume partitions are inevitable and document how every subsystem behaves when links fail.

## Summary
- **Consistency (C)**: Every read returns the latest successful write (linearizability).
- **Availability (A)**: Every request receives a non-error response, even if the data is stale.
- **Partition Tolerance (P)**: The system continues operating despite message loss or delay.
- When a partition occurs, you must pick CP (reject some requests to stay correct) or AP (serve possibly stale data to stay available).

## Decision Framework
1. **Steady State (PACELC)**: Even with healthy links, you still trade latency vs. consistency. Spanner waits on TrueTime for exact ordering; Dynamo favors low latency knowing it may return stale reads.
2. **During Partitions**: CP systems (Spanner, ZooKeeper, etcd) require majority quorum so minority partitions reject writes. AP systems (Cassandra, Dynamo, Riak) accept local writes and reconcile conflicts later.
3. **Tunable Consistency**: Quorum stores expose `R` and `W` knobs. If `R + W > N`, requests behave CP; otherwise they tilt toward AP with better latency.
4. **Business Context**: Payments and inventory typically choose CP; user timelines, analytics dashboards, or logging pipelines often remain AP to preserve availability.

## Design Checklist
- **Conflict Resolution**: Prefer deterministic merges (LWW, CRDTs, domain logic) and ensure operations are idempotent so retries are safe.
- **Client UX**: Surface freshness indicators or disable high-risk actions when replicas diverge.
- **Operational Signals**: Monitor heartbeat latency, quorum success rate, replica divergence, and gossip convergence so partitions are observable.
- **RPO/RTO Clarity**: Document acceptable data loss and downtime so responders know which dial to turn during incidents.
- **Chaos Testing**: Run Jepsen-style experiments that cut links, drop packets, and reorder messages to verify guarantees.

## Beyond CAP
- **PACELC** emphasizes that even without partitions you must trade latency vs. consistency.
- **CALM Theorem** shows that monotonic programs (state only grows) can converge without coordination, a foundation for CRDT data types.
- **Hybrid Architectures** often pair CP control planes (leader election, metadata) with AP data paths (logs, caches). Document the boundary clearly.

## Communication Patterns
- Use precise language with stakeholders ("read-after-write within a single region") instead of vague "strong consistency" claims.
- Explain to support and product teams why certain actions become read-only during network events; tie the decision back to business risk appetite.
- Keep consistency levels explicit in SDKs and logs so you can audit who opted into weaker guarantees.

## Interview Drills
1. Describe how Cassandra can emulate CP semantics for a specific workload and the resulting trade-offs.
2. Propose a customer-facing mitigation plan when conflicting order updates occur in an AP shopping cart service.
3. Outline the validation steps you would run before trusting a vendor's "CP" coordination service for leader election.
