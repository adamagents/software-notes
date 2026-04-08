# Chapter 8: Design a URL Shortener

## Summary

This chapter designs a service that turns long URLs into compact aliases and resolves them at very high read volume. The problem is deceptively small but touches hashing, collision handling, caching, storage efficiency, and abuse prevention.

## Key Ideas

- The core API surface is small: shorten a URL and redirect from a short code.
- Reads dominate writes, so caching redirect lookups is critical.
- Hashing the long URL directly is compact but must handle collisions.
- Generating a unique numeric ID and converting it to base-62 yields short, dense aliases with predictable uniqueness.
- Durable storage maps short codes to original URLs and can also retain metadata such as creation time or expiration.
- Optional features such as custom aliases, analytics, spam controls, and expiration rules shape the final design.

## Interview Takeaway

The strongest answers treat the redirect path as latency-sensitive and operationally simple, while separating write-time concerns like collision avoidance, ID generation, and metadata persistence.
