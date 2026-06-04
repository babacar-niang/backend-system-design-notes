# Transactional Outbox Pattern

The Transactional Outbox Pattern is a reliability pattern used in distributed systems to guarantee consistency between business data stored in a database and events published to a message broker such as Kafka.

It solves one of the most common problems in event-driven architectures: the **dual-write problem**.

---

## Transactional Outbox Flow

![Transactional Outbox Pattern](diagram/outbox-pattern-flow.png)

---

## The Dual-Write Problem

Consider the following implementation:

```text
1. Save payment to database
2. Publish event to Kafka
```

At first glance, this appears straightforward.

However, failures can occur between these two operations.

### Failure Scenario

```text
1. Payment saved successfully
2. Application crashes
3. Kafka event never published
```

Result:

* The payment exists in the database.
* No downstream service receives the event.
* The system becomes inconsistent.

This creates a reliability gap that can be difficult to detect and recover from.

---

## The Solution

Instead of publishing directly to Kafka, the application writes both:

* Business data
* Event data

inside the same database transaction.

```text
1. Create payment
2. Insert payment row
3. Insert outbox event row
4. Commit transaction
```

If the transaction succeeds, both records exist.

If the transaction fails, neither record exists.

This guarantees atomicity.

---

## Processing Flow

Once the transaction is committed:

```text
1. Outbox Poller scans unpublished events
2. Event is published to Kafka
3. Publication succeeds
4. Event is marked as published
```

This separates business operations from message delivery.

---

## Example Database Schema

```sql
CREATE TABLE outbox_events (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id UUID NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL,
    published_at TIMESTAMP
);
```

The table acts as a temporary event store until messages are successfully delivered.

---

## Benefits

### Strong Consistency

Business data and events remain synchronized.

### Reliable Event Delivery

Events cannot be silently lost after a successful database transaction.

### Retry Support

Failed publications can be retried safely.

### Auditability

Every generated event remains traceable within the database.

### Operational Visibility

Stuck or unpublished events can be monitored and investigated.

---

## Common Implementation Variants

### Polling Publisher

A scheduled process periodically scans the outbox table.

Advantages:

* Simple implementation
* Easy to understand
* Database agnostic

Trade-offs:

* Additional polling load
* Slight delivery latency

---

### Change Data Capture (CDC)

Tools such as Debezium stream database changes directly into Kafka.

Advantages:

* Near real-time delivery
* No polling process

Trade-offs:

* Additional infrastructure
* Increased operational complexity

---

## Monitoring Recommendations

A production system should monitor:

* Number of unpublished events
* Event publication latency
* Failed publication attempts
* Kafka publish errors
* Poller execution time

An increasing backlog of unpublished events is often an early indicator of downstream issues.

---

## Common Mistakes

### Publishing Directly After Commit

This reintroduces the dual-write problem.

### Deleting Failed Events

Failed events should remain recoverable.

### Missing Idempotency

Consumers should be designed to handle duplicate deliveries safely.

### Ignoring Monitoring

An outbox table can silently grow if publication failures are not detected.

---

## When To Use

The Transactional Outbox Pattern is recommended whenever business state and event streams must remain consistent.

Typical use cases include:

* Payment platforms
* Banking systems
* Order management systems
* Marketplace platforms
* Financial ledgers
* Event-driven microservices

---

## Design Principle

A distributed system should never rely on a database write and a message publish succeeding independently.

The Transactional Outbox Pattern ensures that business data and business events remain part of the same reliable workflow.
