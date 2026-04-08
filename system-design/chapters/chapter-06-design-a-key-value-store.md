# Chapter 6: Design a Key-Value Store

## Summary

This chapter designs a distributed key-value store in the Dynamo style. The system prioritizes partition tolerance, high availability, and horizontal scalability while accepting eventual consistency in the default case.

## Key Ideas

- Clarify the API first: `get`, `put`, and delete-like operations are enough for the core store.
- Partition data with consistent hashing so capacity can scale horizontally.
- Replicate keys to multiple nodes for durability and availability.
- Use tunable quorums so reads and writes can trade latency for stronger consistency.
- Detect and reconcile conflicts with versioning tools such as vector clocks.
- Rely on gossip-based membership and failure detection instead of a single coordinator.
- Use a log-structured storage engine with an in-memory index and SSTables for efficient writes.
- Background compaction, replica synchronization, and hinted handoff are operational necessities, not extras.

## Interview Takeaway

The chapter is a reminder that a distributed data store is mostly about failure handling: node churn, replica divergence, hinted writes, repair, and consistency semantics matter as much as the happy path.
