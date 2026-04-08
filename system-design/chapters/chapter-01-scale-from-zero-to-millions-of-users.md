# Chapter 1: Scale from Zero to Millions of Users

## Summary

This chapter walks through the classic progression from a single-box application to a multi-tier, globally distributed system. The core lesson is that large-scale systems are rarely invented fully formed; they evolve by isolating bottlenecks and introducing the next layer of indirection only when scale demands it.

## Key Ideas

- Start simple with a single server that handles web traffic, business logic, and persistence.
- Split the web tier and data tier early so they can scale independently.
- Prefer horizontal scaling over vertical scaling once reliability and growth matter.
- Use a load balancer to remove the web-tier single point of failure.
- Add database replication so reads scale out and failures are survivable.
- Introduce a cache tier for hot data and a CDN for static assets to reduce latency and database load.
- Move session state out of web servers so the fleet becomes stateless and autoscalable.
- Add multiple data centers, message queues, logging, metrics, and deployment automation as the system matures.
- Shard the database when a single primary can no longer absorb the write volume or dataset size.

## Interview Takeaway

A strong answer to "how do you scale this system?" should sound incremental. Explain what breaks first, which component you separate next, what trade-off you are accepting, and how each change improves availability, latency, or operability.
