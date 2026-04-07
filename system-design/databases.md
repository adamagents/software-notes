# Database Choices

Choosing a datastore is about matching workload characteristics to consistency, scale, and operational constraints. Good architectures often pair multiple specialized databases rather than searching for a silver bullet.

## Summary
- **Workload First**: Start with access patterns, latency budgets, write amplification, and compliance boundaries.
- **Managed When Possible**: Offload mundane operations (patching, backups, scaling) to managed services unless requirements forbid it.
- **Polyglot Persistence**: Different bounded contexts deserve their own stores, coordinated via events or change-data capture.
- **Operational Rigor**: Monitoring, migrations, and backup drills matter as much as the schema itself.

## Selection Framework
| Question | Examples | Implications |
| --- | --- | --- |
| Access Pattern | Point lookups, range scans, graph traversals | Determine index strategy, doc vs. relational vs. graph |
| Write Profile | Steady, bursty, multi-region writers | Influences storage engine (LSM-tree vs. B-Tree) and replication style |
| Consistency Needs | Strong ACID, read-after-write, eventual | Drives choice between CP, AP, or tunable systems |
| Data Gravity | Size, retention, compliance rules | Impacts storage tiering, encryption, residency |

## Relational Databases
- Offer transactions, joins, referential integrity, and decades of tooling (PostgreSQL, MySQL, SQL Server).
- Scale vertically with replicas; use sharding frameworks (Vitess, Citus, Spanner) when write load exceeds a single primary.
- Schema migrations require discipline: online migrations, feature flags, shadow tables, and rollback scripts.
- Perfect fit for financial ledgers, multi-step workflows, and domains with complex relationships.

## NoSQL Families
| Type | Strengths | Typical Workloads |
| --- | --- | --- |
| Key-Value (DynamoDB, FoundationDB) | Predictable latency, elastic scale | Session stores, catalog lookups, gaming state |
| Document (MongoDB, Couchbase) | Flexible schema, secondary indexes | User profiles, CMS content, product data |
| Wide-Column (Cassandra, Bigtable, HBase) | Massive writes, tunable consistency | Time-series, IoT telemetry, messaging metadata |
| Graph (Neo4j, JanusGraph) | Efficient traversals, path analytics | Recommendations, fraud detection |
| Search Engines (Elasticsearch, OpenSearch) | Relevance scoring, full-text + aggregations | Logs, e-commerce search, analytics over text |

## Polyglot Persistence Patterns
- Partition the domain into bounded contexts; each owns its data model and publishes change events.
- Use CDC (Debezium, Dynamo Streams, GoldenGate) to feed caches, read models, or analytics warehouses.
- Avoid cross-database joins; compose data via APIs or asynchronous pipelines.
- Guardrails: consistent identifiers, schema registry, and lineage catalog so downstream systems know data provenance.

## Multi-Region & High Availability
- **Primary/Replica**: Async replication with defined RPO/RTO; rehearse failovers and understand replica lag visibility.
- **Active-Active**: Requires conflict detection, per-field merges, or CRDTs; best for append-only or commutative workloads.
- **Consensus Stores**: etcd, Spanner, and Yugabyte use quorum protocols for strongly consistent writes at the cost of higher latency.
- **Backups & Recovery**: Automate full + incremental backups, verify PITR windows, and perform restore drills regularly.

## Operational Guardrails
- Monitor slow queries, lock wait time, replication lag, disk utilization, cache hit ratios, and connection pool usage.
- Establish migration playbooks with canary batches, automated rollbacks, and verification queries.
- Control costs via right-sized instances, storage tiering (hot SSD vs. cold object store), and archival policies for historical data.
- Enforce least-privilege access, audit logging, and data-masking strategies for lower environments.

## Interview Drills
1. Compare sharding a relational database versus migrating to Cassandra for a write-heavy workload.
2. Design a multi-tenant database strategy that balances noisy neighbors against operational simplicity.
3. Explain how you would detect and remediate replication lag causing stale reads in a global application.
