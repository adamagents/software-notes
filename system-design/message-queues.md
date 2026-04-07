# Message Queues

## Overview
Message queues and event logs decouple producers from consumers, absorb bursts, and deliver resiliency primitives (retry, dedup, ordering) that synchronous RPC cannot easily provide. Treat the queue as critical infrastructure: version schemas, monitor lag, and design consumers to be idempotent and horizontally scalable.

## Core Challenges
- **Delivery guarantees**: Choosing between at-most-once, at-least-once, and exactly-once semantics affects handler design, storage, and billing.
- **Ordering**: Some workflows require per-entity ordering, demanding carefully chosen partition keys or single-threaded queues.
- **Backpressure**: Slow consumers or downstream outages must propagate upstream without data loss.
- **Schema evolution**: Producers and consumers evolve independently; contracts need registries and compatibility checks.

## Architecture Building Blocks
- **Traditional queues**: SQS, RabbitMQ, and Azure Service Bus offer per-message ACKs, visibility timeouts, and dead-letter queues.
- **Log-based systems**: Kafka, Pulsar, Redpanda retain ordered partitions and let multiple consumer groups read independently.
- **Workflow engines**: Temporal, Cadence, and Step Functions coordinate multi-step processes with retry policies and state tracking.
- **Schemas and registries**: Avro, JSON Schema, or Protobuf definitions stored centrally prevent breaking changes.
- **Control plane**: Provisioning, quota management, encryption keys, and audit logs belong here.

## Design Checklist
- Pick acknowledgement strategy. Visibility timeouts + retry counts (queues) or committed offsets + consumer groups (logs) must be tuned to workload latency.
- Enforce idempotency via application-level keys or dedup tables so at-least-once delivery does not create duplicate side effects.
- Create DLQ policies that include metadata (stack trace, headers, payload checksum) and alert owners automatically.
- Use partition keys tied to business entities (account ID, order ID) when ordering matters; avoid random keys that defeat batching.
- Distinguish between transactional messaging (exactly-once) and analytics/event sourcing (append-only) so retention and storage costs remain predictable.

## Failure Modes and Mitigations
- **Poison messages**: Move to DLQ quickly, cap retry attempts, and provide a replay workflow once the bug is fixed.
- **Runaway backlog**: Track oldest message age; auto-scale consumers, shed noncritical producers, or drop to sampling when lag exceeds SLA.
- **Out-of-order processing**: Add sequence numbers, enforce partitioned consumption, or buffer until missing events arrive.
- **Duplicate deliveries**: Persist idempotency tokens or use transactional outbox patterns to avoid double writes.
- **Consumer crashes**: Externalize checkpoints, use heartbeat/lease mechanisms, and prefer stateless handlers.

## Observability and Operations
- Instrument enqueue/dequeue rates, lag, DLQ volume, ACK latency, consumer error codes, and batch sizes.
- Emit trace spans or log correlation IDs to follow a message across services.
- Provide tooling for selective replays, DLQ inspection, and message redrive with rate limits.
- Run chaos tests that pause partitions, corrupt offsets, or inject malformed payloads to validate recovery scripts.
- Apply access controls, encryption, and audit logging because queues often carry sensitive data.

## Interview Prompts
1. Design a payment workflow where events must be processed exactly once and in order.
2. Explain how you would migrate from a monolithic cron batch to an event-driven pipeline without losing in-flight jobs.
3. Describe the metrics and alerts you need to detect a backlog forming before users notice delays.
