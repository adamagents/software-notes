# Database Choices

Matching data stores to workload characteristics keeps systems simple, fast, and maintainable. Every database decision trades query richness, consistency, cost, and operational effort.

## Requirements Checklist
- **Access Patterns**: Point lookups, range scans, aggregates, graph traversals?
- **Write Profile**: Steady stream vs. large bursts, single-region vs. multi-region writers.
- **Consistency Guarantees**: Strong ACID, read-after-write, or eventual acceptable?
- **Latency Budgets**: Sub-millisecond for user paths vs. seconds for analytics.
- **Operational Constraints**: Managed cloud, self-hosted, air-gapped, compliance requirements.

## Relational Databases
- Offer transactions, joins, and mature tooling (PostgreSQL, MySQL, SQL Server).
- Vertical scaling + read replicas cover most growth; sharding via Vitess/Citus for extreme scale.
- Schema migrations require discipline: use online migrations, feature flags, and rollback scripts.
- Best fit for financial ledgers, inventory, and workflows needing referential integrity.

## NoSQL Families
| Type | Strengths | Typical Workloads |
| --- | --- | --- |
| Key-Value (DynamoDB, FoundationDB) | Predictable latency, horizontal scale | Session storage, IoT metrics |
| Document (MongoDB, Couchbase) | Flexible schema, rich secondary indexes | User profiles, CMS content |
| Wide-Column (Cassandra, HBase) | Massive writes, tunable consistency | Time-series, messaging metadata |
| Graph (Neo4j, JanusGraph) | Efficient traversals, path queries | Recommendations, fraud detection |

## Polyglot Persistence
- Partition the domain into bounded contexts; each owns its database and publishes change events.
- Build authoritative logs (CDC, Debezium, Dynamo Streams) so downstream caches/read models can rebuild.
- Guardrails: avoid cross-database joins via API composition or data pipelines.

## Multi-Region & HA
- **Primary/Replica**: Async replication with defined RPO/RTO; failover runbooks are mandatory.
- **Active-Active**: Requires conflict detection/merging (CRDTs, vector clocks, monotonic counters).
- **Consensus Stores**: etcd/Spanner offer strongly consistent global state at the cost of higher write latency.

## Observability & Operations
- Monitor slow queries, lock wait time, replica lag, disk utilization, and cache hit ratios.
- Automate backups + regular restore drills; verify checksums and PITR windows.
- Enforce connection pooling to prevent thundering herds during traffic spikes.

## Cost Controls
- Right-size instance classes and storage tiers; watch IOPS vs. provisioned baselines.
- Archive cold data to cheaper stores (S3, Glacier) with lifecycle policies.
- For serverless offerings, cap provisioned throughput and use auto-scaling policies with alerts.

## Interview Drills
1. Compare when you would shard a relational database versus migrate to Cassandra.
2. Design a multi-tenant database strategy that balances noisy neighbors against operational simplicity.
3. Explain how you would detect and recover from replication lag causing stale reads.
