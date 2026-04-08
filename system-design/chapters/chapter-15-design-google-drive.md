# Chapter 15: Design Google Drive

## Summary

This chapter designs a cloud file storage and synchronization system. The hard parts are reliable storage, fast cross-device sync, version tracking, and minimizing unnecessary data transfer.

## Key Ideas

- Start by narrowing the scope to upload, download, sync, sharing, revision history, and notifications.
- Separate metadata from blob storage so file contents and file records can scale independently.
- Use chunked uploads and resumable transfers for reliability over unstable networks.
- Sync should be incremental: only changed chunks or deltas should move when possible.
- Keep revision history so users can recover older file versions and resolve conflicts.
- Notifications tell connected clients that a file changed so they can fetch updates promptly.
- Reliability and durability are first-class requirements because silent data loss is unacceptable.

## Interview Takeaway

The chapter teaches that "cloud drive" is not just object storage plus an API. The product depends on metadata correctness, revision semantics, conflict handling, and efficient synchronization across devices.
