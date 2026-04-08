# Chapter 7: Design a Unique ID Generator in Distributed Systems

## Summary

This chapter evaluates several ways to generate unique identifiers at scale and lands on a Snowflake-style solution as the most practical balance of uniqueness, ordering, and throughput.

## Options Considered

- Multi-master auto-increment databases are simple but create coordination pain and poor portability.
- UUIDs avoid coordination but are large, unordered, and index-unfriendly.
- A ticket server centralizes sequencing but becomes an obvious bottleneck and availability risk.
- Snowflake-style IDs split a 64-bit integer into timestamp, machine identity, and per-millisecond sequence.

## Key Ideas

- A good distributed ID should be unique, sortable enough for storage and pagination, and cheap to generate locally.
- Time-based IDs need clock discipline and a plan for clock drift or rollback.
- Worker ID assignment must be coordinated so two generators never emit the same bit pattern.
- Per-node sequence counters solve local collisions within the same timestamp bucket.

## Interview Takeaway

Explain why the winning design is usually the one that removes the database from the hot path while keeping identifiers compact and monotonic enough for downstream systems.
