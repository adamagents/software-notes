# Chapter 12: Design a Chat System

## Summary

This chapter designs a large-scale messaging system with one-to-one chat, small-group chat, online presence, multi-device support, and push notifications. The architecture is built around long-lived connections and careful message-state tracking.

## Key Ideas

- Scope matters: one-to-one chat, group size, latency target, retention policy, and encryption all change the design.
- Sending can start over HTTP, but receiving low-latency messages works better with WebSocket or another persistent bidirectional connection.
- Routing requires a service that knows which devices are online and where each session is connected.
- Store message history durably and keep recent conversations cached for fast sync.
- Presence is its own subsystem because online indicators and last-seen timestamps change frequently.
- Support multiple devices per account by tracking per-device delivery and acknowledgment state.
- Push notifications bridge the gap when recipients are offline or backgrounded.

## Interview Takeaway

Chat design is mostly about connection management and state synchronization. The correct answer is usually the one that explains how messages, presence, and device sessions stay coherent under reconnects and partial failures.
