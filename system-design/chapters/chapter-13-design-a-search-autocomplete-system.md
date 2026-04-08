# Chapter 13: Design a Search Autocomplete System

## Summary

This chapter designs a low-latency service that returns the top suggestions for a typed prefix. The core challenge is serving ranked results within roughly 100 ms while continuously updating popularity data.

## Key Ideas

- The query path must be optimized for prefix lookups and tiny responses.
- A trie is a natural fit because it groups strings by prefix.
- Each trie node can cache the top-k most popular completions under that prefix to avoid scanning the subtree on every request.
- Real-time frequency updates are expensive at high scale, so the production design separates ingestion from periodic rebuilds.
- Normalization rules matter: lowercase-only input is simple, while multilingual input adds tokenization and ranking complexity.
- Availability, cacheability, and regional proximity are important because autocomplete sits directly on the typing path.

## Interview Takeaway

The chapter shows how a conceptually simple feature becomes an indexing problem. The best explanation focuses on how ranking data is maintained without making the read path slow.
