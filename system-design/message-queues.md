# Message Queues & Event-Driven Workflows

Queues decouple producers and consumers, absorb bursts, and provide delivery guarantees that synchronous RPC often cannot match.

## Summary
- **Temporal Decoupling**: Producers keep working even when consumers are offline.
- **Load Smoothing**: Elastic queues buffer spikes so worker fleets can scale gradually.
- **Delivery Guarantees**: Acknowledgment, retry, and dedup semantics guard against data loss.
- **Backpressure**: Feedback loops throttle producers before downstream systems collapse.

## Queue vs. Log Streams
| Feature | Message Queue (SQS, RabbitMQ) | Log Stream (Kafka, Pulsar) |
| --- | --- | --- |
| Consumption | Messages removed once ACKed | Data retained; consumers track offsets |
| Ordering | Per-queue or FIFO subsets | Per-partition ordering |
| Fan-out | Explicit bindings or topics per consumer | Consumer groups read the same partition independently |
| Retention | Minutes to days | Configurable, often long-term |
| Ideal Uses | Task processing, workflows | Event sourcing, CDC, analytics |

## Delivery Semantics
- **At-Least-Once**: Default mode; requires idempotent consumers and dedupe keys.
- **At-Most-Once**: ACK before processing; acceptable only when drops are tolerable.
- **Exactly-Once**: Combine idempotent producers, transactional commits (Kafka EOS, Pulsar transactions), and deterministic consumers.
- **Poison Message Handling**: Dead-letter queues capture failures with metadata for replay or debugging.

## Consumer Design Patterns
- **Competing Consumers**: Multiple workers share the queue; monitor lag and rebalance partitions regularly.
- **Retry Policies**: Use exponential backoff with jitter and capped attempts to avoid infinite loops.
- **Scheduling & Cron**: Delay queues or scheduled topics trigger work at a future timestamp.
- **State Management**: Keep handlers stateless; store checkpoints externally for crash recovery.

## Operational Guardrails
- Backpressure by throttling producers when backlog age or queue depth exceed thresholds.
- Apply adaptive batching (combine N messages) to raise throughput while honoring latency SLAs.
- Separate priority lanes (high vs. normal) to prevent critical jobs from starving.
- Enforce schema/contracts through registries (Avro, JSON Schema, Protobuf) so new fields stay compatible.

## Failure Modes & Mitigations
- **Poison Messages**: Route to DLQ with context, alert operators, and provide replay tooling.
- **Slow Consumers**: Auto-scale workers, shard partitions differently, or shed load via circuit breakers.
- **Out-of-Order Processing**: Use partition keys for per-entity ordering, or add sequence numbers to detect conflicts.
- **Duplicate Deliveries**: Embed idempotency keys and persist processing results alongside checkpoints.

## Observability & Operations
- Track enqueue/dequeue rate, oldest message age, consumer lag, DLQ volume, and ACK latency.
- Instrument distributed tracing with message IDs to follow events across services.
- Capacity-plan for the largest backlog you are willing to drain within recovery objectives.
- Provide admin tooling for replays, priority overrides, and dead-letter inspection.

## Interview Drills
1. Design a workflow that guarantees ordering for payments that must be processed sequentially.
2. Describe how you would migrate a monolith to an event-driven architecture without losing in-flight jobs.
3. Explain how to keep a single slow consumer from blocking a queue shared by many fast consumers.
