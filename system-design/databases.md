# Database Choices

Healthy systems pick database technologies that match workload characteristics instead of chasing trends. Every choice trades durability, latency, query flexibility, and operational complexity.

## Decision Dimensions
| Dimension | Questions to Ask |
| --- | --- |
| Access Pattern | Point lookups, range scans, graph traversals, analytical aggregations? |
| Consistency | Strict ACID, read-after-write, eventual consistency acceptable? |
| Latency Targets | Sub-millisecond, single-digit ms, or analytics-scale seconds? |
| Scale Profile | Steady growth, unpredictable bursts, write-heavy vs. read-heavy? |
| Schema Evolution | Do fields evolve frequently or need ad-hoc columns? |
| Operational Model | Managed service availability, self-hosted constraints, multi-region? |

## SQL Databases
- Strong consistency, transactions, rich indexing, relational joins.
- Vertical scaling plus read replicas; sharding adds complexity but well-trodden (e.g., Vitess, Citus).
- Best for financial systems, inventory, strong referential integrity.
- Tune with normalized schemas, query plans, partitioning, and caching of heavy joins.

## NoSQL Families
- **Key-Value Stores** (DynamoDB, Riak): Predictable latency, horizontal scaling, but limited querying beyond primary key.
- **Document Stores** (MongoDB, Couchbase): Flexible JSON-like records, indexes on nested fields, eventual or tunable consistency.
- **Columnar Wide-Column Stores** (Cassandra, HBase): Optimized for large sequential writes and wide datasets; rely on denormalization and composite keys.
- **Graph Databases** (Neo4j, JanusGraph): Efficient traversals and path queries, useful for recommendation or fraud graphs.

## Polyglot Persistence
- Use multiple databases per bounded context, but **own the data flow contracts** (CDC pipelines, change schemas, eventual reconciliation playbooks).
- Keep authoritative source of truth clear; caches or read models should rebuild from upstream logs.

## Multi-Region Considerations
- Define primary vs. active-active replication. Cross-region writes require conflict resolution (CRDTs, last-write-wins, application-specific merges).
- WAN latency pushes teams toward local writes with async replication; be explicit about RPO/RTO metrics.

## Operational Playbook
- Implement schema migration tooling (online migrations, feature flags for new fields).
- Monitor lag for replicas, storage utilization, slow query logs.
- Backup + restore drills: test PITR, verify checksums, isolate restored clusters for validation before promoting.

## Interview Prompts
1. When would you choose a relational DB plus caching over a document store for user profiles?
2. How would you shard a multi-tenant database while still enabling analytics per tenant?
3. What failure modes appear when replica lag grows beyond SLA, and how do you mitigate them?
