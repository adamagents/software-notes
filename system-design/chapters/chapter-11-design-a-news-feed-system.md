# Chapter 11: Design a News Feed System

## Summary

This chapter breaks the product into two flows: publishing a post and building the feed shown to readers. The main design trade-off is between pushing new posts into follower feeds ahead of time and pulling posts on demand at read time.

## Key Ideas

- Clarify the scope: post creation, feed retrieval, ranking model, friend limits, and media support.
- Feed publishing persists the post, updates caches, and fans out the new content to followers.
- Feed retrieval reads a precomputed or partially computed feed from cache for low-latency reads.
- Fan-out on write works well for ordinary users because reads are cheap after publication.
- Fan-out on read is better for celebrities or extreme follower counts where write amplification becomes too expensive.
- Metadata and feed entries usually live in separate stores because their access patterns differ.
- Caching is central on both the post side and the assembled feed side.

## Interview Takeaway

The chapter's core lesson is the push-vs-pull trade-off. Good answers explain when to precompute feeds, when to compute on demand, and how to treat high-fan-out users differently.
