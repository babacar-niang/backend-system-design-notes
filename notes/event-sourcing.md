# Event Sourcing

Event Sourcing is an architectural pattern where state changes are stored as a sequence of immutable events rather than persisting only the latest state.

Instead of storing the current state directly, the system stores the complete history of how that state was produced.

This approach provides a full audit trail and enables powerful capabilities such as replay, temporal queries, and historical reconstruction.

---

## Event Sourcing Flow

![Event Sourcing Flow](diagram/event-sourcing-flow.png)

---

## Traditional State-Based Model

Most applications store only the current state.

Example:

```text
Account Balance = 100,000 XOF
```

The previous operations that produced this balance are lost or only partially recorded.

Questions become difficult to answer:

* How did the balance reach this value?
* What transactions occurred yesterday?
* What was the balance last month?

---

## Event-Sourced Model

Instead of storing the current state, the system stores all domain events.

Example:

```text
AccountCreated
MoneyDeposited
MoneyWithdrawn
FeeApplied
```

The current state is reconstructed by replaying events in order.

---

## Example Event Stream

```json id="8xgs3q"
[
  {
    "type": "ACCOUNT_CREATED",
    "balance": 0
  },
  {
    "type": "MONEY_DEPOSITED",
    "amount": 100000
  },
  {
    "type": "MONEY_WITHDRAWN",
    "amount": 25000
  }
]
```

Current balance:

```text id="95twxg"
100000 - 25000 = 75000
```

The balance is derived rather than stored directly.

---

## How State Reconstruction Works

The application replays events sequentially:

```text id="40npj7"
AccountCreated
      ↓
Balance = 0

MoneyDeposited
      ↓
Balance = 100000

MoneyWithdrawn
      ↓
Balance = 75000
```

The final state is a projection of the event history.

---

## Read Models and Projections

Rebuilding state from millions of events for every query is inefficient.

Most Event Sourcing systems create projections.

Example:

```text id="33ot77"
Event Store
      ↓
Projector
      ↓
Account Balance Read Model
```

The read model is optimized for querying while the event store remains the source of truth.

This approach is commonly combined with CQRS (Command Query Responsibility Segregation).

---

## Benefits

### Complete Audit Trail

Every state transition is preserved.

Example:

```text id="o5m3ev"
Who changed the balance?
When?
Why?
```

The answers are stored in the event stream.

---

### Historical Reconstruction

The system can reconstruct state at any point in time.

Example:

```text id="m0q6uq"
Balance on:
- Yesterday
- Last week
- Last year
```

without requiring special backup tables.

---

### Event Replay

New features can be built by replaying historical events.

Example:

```text id="4rb6uh"
New Analytics Service
      ↓
Replay Events
      ↓
Build Historical Reports
```

No data migration is required.

---

### Strong Fit for Financial Systems

Financial platforms often require:

* Auditability
* Traceability
* Compliance
* Historical analysis

Event Sourcing naturally supports these requirements.

---

### Debugging Complex Workflows

The entire business journey can be reconstructed from recorded events.

This makes root cause analysis significantly easier.

---

## Trade-Offs

### Increased Complexity

Event Sourcing introduces additional architectural concepts:

* Event Store
* Projections
* Replay mechanisms
* Event versioning

A simple CRUD application is often easier to maintain.

---

### Event Versioning

Events evolve over time.

Example:

```json id="5pj4xw"
{
  "eventVersion": 2,
  "eventType": "MONEY_DEPOSITED"
}
```

Backward compatibility becomes an important concern.

---

### Projection Maintenance

Read models must remain synchronized with the event stream.

Monitoring and recovery strategies become critical.

---

### Operational Tooling

Without proper tooling:

* Event inspection
* Replay
* Debugging

can become difficult.

---

## Common Use Cases

Event Sourcing works particularly well for:

* Financial ledgers
* Payment systems
* Banking platforms
* Trading systems
* Audit-heavy applications
* Compliance-sensitive workflows
* Order lifecycle tracking

Examples:

```text id="jv4m1k"
AccountOpened
FundsDeposited
FundsTransferred
InterestApplied
AccountClosed
```

Every operation becomes part of the permanent business history.

---

## When Not To Use It

Event Sourcing is often unnecessary for:

* Simple CRUD applications
* Internal administration tools
* Small content management systems
* Low-audit business processes

If historical reconstruction is not a requirement, traditional persistence is usually simpler.

---

## Common Mistakes

### Storing Technical Events

Events should describe business actions.

Good:

```text id="2tx7h7"
PaymentCompleted
```

Bad:

```text id="9j74kt"
UpdatePaymentRow
```

---

### Ignoring Versioning

Event schemas inevitably evolve.

Versioning should be planned from the beginning.

---

### Using Event Sourcing Everywhere

Not every service requires an event store.

The complexity should be justified by business value.

---

### Treating Read Models as the Source of Truth

The event stream remains the authoritative record.

Read models are projections.

---

## Event Sourcing vs Traditional CRUD

| Aspect             | CRUD            | Event Sourcing |
| ------------------ | --------------- | -------------- |
| Current State      | Stored directly | Reconstructed  |
| Audit Trail        | Limited         | Complete       |
| Historical Queries | Difficult       | Natural        |
| Complexity         | Low             | High           |
| Replay Capability  | No              | Yes            |
| Compliance Support | Moderate        | Strong         |

---

## Design Principle

Event Sourcing should solve a business problem, not an architectural curiosity.

When auditability, traceability, historical reconstruction, and replay capabilities are core requirements, Event Sourcing becomes a powerful foundation for building reliable financial and distributed systems.

When those requirements do not exist, a traditional CRUD model is often the better choice.
