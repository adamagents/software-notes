# Chapter 14: Design YouTube

## Summary

This chapter designs a video platform around two expensive workflows: uploads and playback. The architecture leans heavily on cloud object storage, CDNs, asynchronous transcoding, and metadata services because video workloads are too large to handle like ordinary web content.

## Key Ideas

- Clarify product scope first: upload, playback, adaptive quality selection, device coverage, and file-size limits.
- Store original uploads durably in blob storage rather than on app servers.
- Use asynchronous transcoding pipelines to generate multiple encodings and resolutions for adaptive streaming.
- Put playback behind a CDN because video delivery dominates both latency and cost.
- Separate metadata, user-facing APIs, and media-processing workers so uploads do not block the control plane.
- Encrypt content, validate uploads, and tolerate resumable or chunked upload flows.
- Cost matters as much as scale; CDN egress dominates the economics, so cache-hit ratio and transcoding policy affect business viability.

## Interview Takeaway

A YouTube-style design should highlight media pipelines, not just generic web architecture. Upload processing, format conversion, storage layout, and CDN distribution are the heart of the system.
