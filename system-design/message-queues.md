# Message Queues & Event-Driven Architecture

Message queues decouple producers and consumers, absorb bursts, and provide delivery guarantees that make distributed workflows reliable.

## Queue vs. Stream
| Feature | Message Queue (SQS, RabbitMQ) | Log/Stream (Kafka, Pulsar) |
| --- | --- | --- |
| Consumption Model | Message removed once ACKed | Consumers track offsets; data retained |
| Ordering | Per-queue or per-partition | Per-partition | 
| Fan-out | Often via explicit bindings | Native via consumer groups |
| Retention | Short-lived | Configurable, days to forever |
| Use Cases | Task processing, workflows | Event sourcing, analytics, CDC |

## Delivery Semantics
- **At-Least-Once**: Default; requires idempotent consumers to tolerate duplicates.
- **At-Most-Once**: ACK before work, fastest but may drop messages.
- **Exactly-Once**: Requires transactional semantics or deterministic deduplication (Kafka with idempotent producers + transactional consumer groups).

## Designing Consumers
- Keep handlers stateless when possible; store progress externally.
- Use visibility timeouts / acknowledgments to allow retries while preventing double processing.
- Scale horizontally via competing consumers; monitor lag per consumer group.
- Implement poison message handling (dead-letter queues) to quarantine bad events.

## Backpressure & Flow Control
- Rate-limit producers or apply adaptive batching when queue depth exceeds thresholds.
- Consumers should checkpoint frequently to avoid replaying large batches after restarts.
- For streaming platforms, use pull-based fetch with max bytes to prevent memory blowups.

## Schema & Contracts
- Version events using schemas (Avro/JSON Schema/Protobuf) and register them in a schema registry.
- Maintain backward compatibility (additive changes) and document semantic meaning of fields.

## Observability
- Track enqueue/dequeue rate, oldest message age, consumer lag, DLQ volume.
- Correlate message IDs through distributed tracing to debug end-to-end latency.

## Interview Prompts
1. How would you guarantee order for payments that must be processed sequentially?
2. Explain how you would migrate from a monolith to event-driven microservices without losing data.
3. How do you prevent a slow consumer from causing unbounded queue growth?
