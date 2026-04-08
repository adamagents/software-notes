# Chapter 5: Design Consistent Hashing

## Summary

This chapter explains how to distribute keys across nodes while minimizing remapping when machines are added or removed. The central win is stability under cluster churn.

## Key Ideas

- Traditional modulo hashing remaps most keys whenever the number of servers changes.
- Consistent hashing places both servers and keys on a ring and assigns each key to the next server clockwise.
- Adding or removing a node affects only a narrow slice of keys instead of the entire keyspace.
- Virtual nodes smooth out imbalance by giving each physical machine multiple ring positions.
- The design must consider skew, rebalancing cost, and failure recovery.
- Replication and rack awareness sit on top of the ring rather than replacing it.

## Interview Takeaway

Consistent hashing is usually introduced to solve resharding pain. A strong answer should explain not only the ring but also why virtual nodes are necessary and what happens during node failure.
