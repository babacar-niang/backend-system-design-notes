# Idempotency in Payment Systems

Idempotency is a critical reliability mechanism in payment systems that ensures the same request can be safely retried without creating duplicate financial transactions.

In distributed systems, network interruptions, client timeouts, or gateway failures can cause clients to resend requests. Without idempotency protection, a single payment attempt could result in multiple charges.

---

## Payment Idempotency Flow

![Payment Idempotency Flow](diagram/payment-idempotency-flow.png)

---

## Why Idempotency Matters

Consider the following scenario:

1. A customer submits a payment request.
2. The payment is successfully created.
3. The response is lost due to a network timeout.
4. The client retries the request.

Without idempotency, the second request may create an entirely new payment.

For financial systems, duplicate payments are often more damaging than temporary service unavailability.

---

## Example Request

```http
POST /payments
Idempotency-Key: 8b1f2c7a
Content-Type: application/json

{
  "amount": 50000,
  "currency": "XOF",
  "receiverId": "merchant-001"
}
```

The server treats multiple requests with the same idempotency key as a single operation.

---

## Implementation Strategy

The server stores:

* The idempotency key
* The original response
* The HTTP status code
* Creation metadata

When a duplicate request arrives:

1. The key is searched.
2. If it exists, the stored response is returned.
3. No new payment is created.

This guarantees exactly-once behavior from the client's perspective.

---

## Example Database Schema

```sql
CREATE TABLE idempotency_keys (
    id UUID PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE NOT NULL,
    response_body JSONB,
    status_code INT,
    created_at TIMESTAMP NOT NULL
);
```

A unique database constraint provides strong protection against concurrent duplicate requests.

---

## Common Approaches

### PostgreSQL-Based Idempotency

The simplest and most reliable solution.

Advantages:

* Strong consistency
* ACID guarantees
* Minimal infrastructure
* Easy operational management

Typical implementation:

```sql
CREATE UNIQUE INDEX idx_idempotency_key
ON idempotency_keys(idempotency_key);
```

---

### Redis-Based Idempotency

Useful in extremely high-throughput environments.

Advantages:

* Very low latency
* Reduced database load

Trade-offs:

* Additional infrastructure
* Expiration management
* Potential consistency challenges

For most payment platforms, PostgreSQL is sufficient.

---

## Best Practices

* Require an `Idempotency-Key` for payment creation endpoints.
* Persist the original response body.
* Store the original HTTP status code.
* Protect keys using unique database constraints.
* Apply expiration policies for stale keys.
* Log idempotency hits for observability.
* Ensure idempotency checks occur before business processing.

---

## Common Mistakes

### Relying on Client Retries

Clients cannot guarantee exactly-once execution.

Idempotency must be enforced server-side.

### Using Request Hashes Only

Different requests may produce identical hashes depending on implementation.

A dedicated idempotency key is more reliable.

### Not Storing the Original Response

Returning a different response for retries can create inconsistent client behavior.

The original response should always be replayed.

---

## Design Principle

In payment systems, duplicate execution is often more dangerous than delayed execution.

A payment request should be processed once, regardless of how many times the client retries it.

Idempotency is one of the fundamental building blocks of reliable financial infrastructure.
