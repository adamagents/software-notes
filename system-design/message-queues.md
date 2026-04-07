# Message Queues & Event-Driven Workflows

Queues decouple producers and consumers, absorb bursts, and deliver reliability guarantees that synchronous RPC often cannot match.

## Goals
- **Temporal Decoupling**: Producers keep working even when consumers are offline.
- **Load Smoothing**: Elastic queues buffer spikes so worker fleets scale gradually.
- **Delivery Guarantees**: ACK/retry semantics guard against data loss.
- **Backpressure**: Feedback loops prevent runaway enqueueing.

## Queue vs. Log Streams
| Feature | Message Queue (SQS, RabbitMQ) | Log Stream (Kafka, Pulsar) |
| --- | --- | --- |
| Consumption | Messages removed once ACKed | Data retained; consumers track offsets |
| Ordering | Per-queue or FIFO subsets | Per-partition ordering |
| Fan-out | Explicit bindings for each consumer | Consumer groups read same partition independently |
| Retention | Minutes to days | Configurable, often long-term |
| Ideal Uses | Task processing, workflows | Event sourcing, analytics, CDC |

## Delivery Semantics
- **At-Least-Once**: Default; require idempotent consumers and dedupe keys.
- **At-Most-Once**: ACK before processing; acceptable only when drops are tolerable.
- **Exactly-Once**: Combine idempotent producers + transactional commits (Kafka EOS, Pulsar transactions) plus deterministic consumers.

## Consumer Design Patterns
- **Competing Consumers**: Scale horizontally; monitor lag per consumer group.
- **Dead-Letter Queues**: Quarantine poison messages with metadata for replay/debugging.
- **Retry Policies**: Exponential backoff, jitter, max-attempt caps to avoid infinite loops.
- **State Management**: Keep handlers stateless; store checkpoints externally for crash recovery.

## Backpressure Strategies
- Dynamic producer throttling when queue depth exceeds thresholds.
- Adaptive batching (combine N messages) to improve throughput while staying within latency SLAs.
- For streaming logs, use fetch limits (bytes + wait time) so slow consumers do not OOM.

## Schema & Contracts
- Enforce schemas via registries (Avro/JSON Schema/Protobuf) with compatibility checks.
- Version fields additively; communicate deprecations well before removal.
- Document semantic meaning (e.g., "idempotency key = order_id") to make consumers safe.

## Observability & Operations
- Track enqueue/dequeue rate, oldest message age, consumer lag, DLQ volume, and ACK latency.
- Instrument tracing with message IDs to follow events across services.
- Capacity plan for the maximum backlog you can drain within recovery time objectives.

## Interview Drills
1. Design a workflow that guarantees ordering for payments that must be processed sequentially.
2. Describe how you would migrate a monolith to an event-driven architecture without losing in-flight jobs.
3. Explain how to keep a single slow consumer from blocking a queue shared by many fast consumers.
