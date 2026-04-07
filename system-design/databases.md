# Databases

## Overview
Selecting a database is an exercise in matching workload traits to engines, not picking a fashionable technology. Capture access patterns, latency budgets, transactions, retention, compliance, and operational staffing before choosing. Production-ready systems pair the right datastore with governance around schema changes, migrations, resilience testing, and observability.

## Core Challenges
- **Workload diversity**: OLTP, analytics, graph traversals, full-text search, and time-series workloads have different index and storage needs.
- **Scaling writes**: Single primaries saturate quickly; sharding, consensus replication, and log shipping introduce complexity.
- **Consistency expectations**: Some domains require serializable transactions, while others tolerate eventual or session-level guarantees.
- **Operations and compliance**: Backups, PITR, encryption, auditing, and data masking must be built into the lifecycle.

## Architecture Building Blocks
- **Relational engines**: PostgreSQL, MySQL, SQL Server, and managed variants excel at normalized schemas, joins, and ACID guarantees.
- **NoSQL families**: Key-value stores serve constant-time lookups, document stores handle flexible schemas, wide-column stores absorb petabyte-scale writes, and graph databases optimize traversals.
- **Search engines**: Elasticsearch/OpenSearch add relevance scoring, aggregations, and log analytics atop inverted indexes.
- **Polyglot persistence**: Bounded contexts own their database but exchange data through CDC, events, or data warehouses.
- **Data governance layer**: Schema registry, migration tooling, and data catalogs keep teams aligned on definitions and ownership.

## Design Checklist
- Start from queries. Identify the top read and write paths, access frequency, and latency SLA. Use that to size indexes, partitions, and caches.
- Prefer managed services unless regulatory or latency constraints demand self-hosting.
- Define migration playbooks: online schema changes, backfills, verification queries, and rollback switches.
- Separate transactional and analytics loads either via read replicas, CDC-fed warehouses, or materialized views.
- Encrypt data at rest and in transit, rotate credentials automatically, and enforce least-privilege access.

## Scaling and Resilience
- **Read scaling**: Use replicas behind read balancers, but expose replica lag to callers so stale reads are understood.
- **Write scaling**: Partition by tenant, geography, or workload characteristics; leverage coordinator frameworks (Vitess, Citus) when domain logic forbids manual sharding.
- **Multi-region**: Decide between active-passive, active-active with conflict resolution, or globally consistent stores like Spanner or Yugabyte.
- **Backups**: Automate full and incremental backups, store them off-site, and rehearse restore drills frequently.
- **Schema evolution**: Use forward-compatible migrations (expand, backfill, contract) and guard deploys with automated checks.

## Observability and Operations
- Monitor slow queries, lock waits, deadlocks, replication lag, buffer cache hit ratio, disk saturation, and connection pool health.
- Include database spans in distributed tracing to attribute latency to query execution vs. network hops.
- Maintain runbooks for failover, parameter tuning, and incident commands. Tie them to dashboards for quick navigation.
- Tag data sets with owners, retention policies, and sensitivity levels so auditing and deletion requests can be fulfilled quickly.
- Use cost governance: track storage tier usage, query costs, and idle replicas to prevent bill shock.

## Interview Prompts
1. Design a multi-tenant SaaS data strategy that balances noisy neighbors with operational simplicity.
2. Compare sharding a monolithic relational database versus adopting Cassandra for a write-heavy workload. What new failure modes appear?
3. Outline a migration path that moves reporting queries off the primary OLTP database without blocking feature work.
