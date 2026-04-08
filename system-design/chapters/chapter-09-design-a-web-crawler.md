# Chapter 9: Design a Web Crawler

## Summary

This chapter builds a scalable crawler that discovers pages, respects politeness constraints, avoids duplicates, and feeds downstream indexing pipelines. Most of the design complexity comes from operating safely at internet scale rather than from fetching HTML itself.

## Key Ideas

- Start by scoping the crawl: target size, file types, freshness requirements, and domains.
- Maintain a URL frontier that prioritizes work while preventing hot domains from being hammered.
- Deduplicate URLs aggressively so the system does not waste bandwidth or storage.
- Respect `robots.txt`, crawl-delay behavior, and per-host politeness budgets.
- Use DNS resolution, fetchers, parsers, and storage as separate horizontally scalable stages.
- Canonicalize URLs before enqueueing them so semantically identical links collapse to one task.
- Persist fetched content, metadata, and crawl state independently so the system can resume cleanly after failures.

## Interview Takeaway

A crawler answer is strongest when it balances throughput with internet etiquette. Freshness, fairness, dedupe, and fault recovery are as important as raw fetch speed.
