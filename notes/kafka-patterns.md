# Kafka Patterns for Backend Systems

Apache Kafka is one of the most widely used platforms for building scalable, event-driven systems.

It enables services to communicate asynchronously while maintaining reliability, ordering guarantees, and fault tolerance.

Kafka is particularly valuable in financial systems where business workflows span multiple services and must remain resilient to failures.

---

## Kafka Retry and Dead Letter Queue Flow

![Kafka Retry and DLQ Flow](diagram/kafka-retry-and-dlq-flow.png)

---

## Why Kafka?

Traditional synchronous communication creates tight coupling between services.

```text
Payment Service
    ↓
Notification Service
    ↓
Ledger Service
    ↓
Fraud Service
```

A failure in any downstream dependency can impact the entire request flow.

Kafka introduces asynchronous communication:

```text
Payment Service
      ↓
    Kafka
      ↓
 ┌─────────────┐
 │ Ledger      │
 │ Notification│
 │ Fraud       │
 └─────────────┘
```

This allows services to evolve and scale independently.

---

## Common Use Cases

Kafka is frequently used for:

* Payment processing
* Order management
* Audit logging
* Notification delivery
* Data synchronization
* Fraud detection
* Event sourcing
* Analytics pipelines
* Real-time monitoring

---

## Producer Pattern

Producers should publish meaningful business events rather than technical events.

Good example:

```json
{
  "eventId": "2d7f4c1e",
  "eventType": "PAYMENT_CREATED",
  "paymentId": "pay_123",
  "amount": 50000,
  "currency": "XOF",
  "createdAt": "2026-06-04T10:00:00Z"
}
```

Poor example:

```json
{
  "action": "insert_payment_row"
}
```

Business events communicate domain intent.

Technical events expose implementation details.

---

## Consumer Pattern

Consumers should be designed to be idempotent.

Duplicate event delivery is expected in distributed systems.

```text
If an event is processed multiple times,
the final business state should remain correct.
```

Examples:

* Ignore duplicate payment events.
* Prevent duplicate notifications.
* Avoid double ledger entries.

Idempotency is a key requirement for reliable Kafka consumers.

---

## Retry Strategy

Not all failures should immediately result in message loss.

Transient failures are common:

* Network interruptions
* Temporary database outages
* External API failures
* Infrastructure restarts

A retry mechanism provides resilience against these issues.

Typical flow:

```text
Main Topic
    ↓
Consumer
    ↓
Failure
    ↓
Retry Topic
    ↓
Consumer
```

Retries are usually configured with exponential backoff to reduce pressure on failing systems.

---

## Dead Letter Queue (DLQ)

Messages that continue failing after the maximum retry count should be redirected to a Dead Letter Queue.

Benefits:

* Prevents poison messages from blocking consumers
* Preserves failed events for investigation
* Supports replay after fixes
* Improves operational visibility

Typical flow:

```text
Main Topic
    ↓
Retry Topic
    ↓
Retry Topic
    ↓
Dead Letter Queue
```

A DLQ should never be ignored.

A growing DLQ is often an indicator of a deeper system issue.

---

## Partitioning Strategy

Partitioning affects both scalability and ordering guarantees.

For payment systems:

```text
Partition Key = paymentId
```

Benefits:

* All events for a payment remain ordered
* Related events are processed sequentially
* Consumer logic becomes simpler

Example:

```text
PAYMENT_CREATED
PAYMENT_PROCESSING
PAYMENT_COMPLETED
```

All events for the same payment should land in the same partition.

---

## Event Versioning

Events evolve over time.

Versioning prevents breaking existing consumers.

Example:

```json
{
  "eventVersion": 2,
  "eventType": "PAYMENT_CREATED"
}
```

Common approaches:

* Version field inside the event
* Topic versioning
* Schema Registry

Backward compatibility should be a design goal.

---

## Consumer Lag Monitoring

Consumer lag represents the difference between produced messages and consumed messages.

A growing lag may indicate:

* Slow consumers
* Infrastructure bottlenecks
* Processing failures
* Capacity issues

Monitoring lag is one of the most important operational metrics for Kafka systems.

---

## Best Practices

* Publish business events instead of technical events.
* Design consumers to be idempotent.
* Use retry topics for transient failures.
* Use DLQs for unrecoverable failures.
* Monitor consumer lag continuously.
* Apply event versioning.
* Protect sensitive data.
* Use partition keys intentionally.
* Avoid large event payloads.

---

## Common Mistakes

### Synchronous Thinking

Kafka is not a remote procedure call.

Services should not expect immediate responses.

### Missing Idempotency

Duplicate deliveries are normal.

Consumers must handle them safely.

### Ignoring DLQs

A DLQ without monitoring quickly becomes a hidden failure queue.

### Poor Partitioning

Bad partition keys can create hotspots and ordering issues.

### Publishing Sensitive Data

Events are often retained for long periods and may be consumed by multiple systems.

Sensitive information should be minimized.

---

## Design Principle

Kafka should be used to communicate business events, not implementation details.

A well-designed event stream enables independent services to collaborate reliably while remaining loosely coupled, scalable, and resilient to failure.
