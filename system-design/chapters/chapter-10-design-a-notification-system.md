# Chapter 10: Design a Notification System

## Summary

This chapter designs a system that sends notifications across multiple channels such as mobile push, SMS, and email. The architecture emphasizes channel isolation, retries, user preferences, and protection against provider failures.

## Key Ideas

- Clarify channel mix first because mobile push, SMS, and email have different delivery semantics and vendors.
- Split the system into an API layer, template/rendering logic, preference management, and channel-specific workers.
- Queue notification jobs so traffic spikes do not overload downstream providers.
- Store user settings, device tokens, templates, and delivery logs separately.
- Make retries explicit and quarantine poison jobs or invalid tokens.
- Track opt-in status, rate limits, and user-level suppression rules to avoid noisy or non-compliant behavior.
- Prefer asynchronous fan-out to external providers so request latency stays low.

## Interview Takeaway

The interesting part of notification design is not sending one message. It is handling fan-out, retries, user preferences, idempotency, and third-party provider instability without dropping critical alerts.
