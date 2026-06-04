# Rate Limiting

Rate limiting is a critical protection mechanism used to safeguard backend systems against abuse, overload, accidental traffic spikes, and denial-of-service scenarios.

Modern platforms use rate limiting to preserve service availability, protect downstream dependencies, and ensure fair resource allocation among clients.

---

## Token Bucket Rate Limiting

![Token Bucket Rate Limiting](diagram/token-bucket-rate-limiting.png)

---

## Why Rate Limiting Matters

Without protection mechanisms, a single client can overwhelm critical infrastructure:

* APIs
* Databases
* Payment providers
* Authentication services
* Search systems
* Message queues
* Background workers

Even legitimate traffic spikes can cause cascading failures across distributed systems.

Rate limiting acts as a first line of defense.

---

## Common Use Cases

Rate limiting is commonly applied to:

* Public APIs
* Authentication endpoints
* Payment creation endpoints
* Webhook receivers
* Search APIs
* File upload services
* AI inference endpoints
* Third-party integrations

Some endpoints require stricter controls than others.

For example:

```text id="9rfj4y"
GET /products
→ moderate limits

POST /payments
→ strict limits

POST /login
→ very strict limits
```

---

## Common Rate Limiting Algorithms

### Fixed Window

The simplest implementation.

Example:

```text id="m6o2ls"
100 requests per minute
```

Advantages:

* Easy to implement
* Low resource usage

Trade-offs:

* Can allow bursts at window boundaries

Example:

```text id="r7bvlh"
100 requests at 12:00:59
100 requests at 12:01:01
```

Effectively:

```text id="6c0k4v"
200 requests in 2 seconds
```

---

### Sliding Window

Tracks requests across a rolling time period.

Advantages:

* More accurate
* Smoother enforcement

Trade-offs:

* Higher implementation complexity
* Additional storage requirements

---

### Token Bucket

The most commonly used algorithm in modern API platforms.

A bucket contains a fixed number of tokens.

Each request consumes one token.

Tokens are replenished continuously over time.

If no token is available:

```text id="20v9tr"
HTTP 429 Too Many Requests
```

Benefits:

* Supports short bursts
* Maintains long-term request limits
* Easy to scale

This approach is used by many API gateways and cloud providers.

---

## Example Response

When limits are exceeded:

```http id="n6i8vd"
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

The client is informed when it may retry.

---

## Standard Rate Limiting Headers

```http id="gqmvz8"
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 60
```

Meaning:

| Header                | Description                              |
| --------------------- | ---------------------------------------- |
| X-RateLimit-Limit     | Maximum allowed requests                 |
| X-RateLimit-Remaining | Remaining requests in the current window |
| X-RateLimit-Reset     | Time until the limit resets              |

These headers improve client behavior and reduce unnecessary retries.

---

## Where To Apply Rate Limiting

### API Gateway

The preferred location.

Benefits:

* Centralized enforcement
* Reduced application complexity
* Lower infrastructure costs

Examples:

* Kong
* NGINX
* AWS API Gateway
* Envoy
* Spring Cloud Gateway

---

### Authentication Endpoints

Protects against:

* Credential stuffing
* Password spraying
* Brute-force attacks

Examples:

```text id="xyiqs4"
/login
/register
/reset-password
```

---

### Payment Endpoints

Protects financial infrastructure from accidental duplicate submissions and abusive traffic.

Examples:

```text id="h52evc"
/payments
/transfers
/payouts
```

---

### Webhook Receivers

Prevents external systems from overwhelming backend services.

---

### Expensive Operations

Endpoints involving:

* Complex database queries
* AI model inference
* External API calls
* Large file processing

should often have stricter limits.

---

## Distributed Systems Considerations

Rate limiting becomes more challenging when applications run on multiple instances.

Example:

```text id="54yeev"
API Instance A
API Instance B
API Instance C
```

Local counters no longer work reliably.

Common solutions:

* Redis
* Distributed caches
* API Gateway enforcement

Redis is one of the most popular implementations for distributed rate limiting.

---

## Monitoring Recommendations

Track:

* Total requests
* Rejected requests
* 429 response rate
* Top consumers
* Token consumption rate

Sudden increases in rejected requests may indicate:

* Abuse attempts
* Misconfigured clients
* Traffic spikes
* Platform incidents

---

## Best Practices

* Apply limits as close to the edge as possible.
* Use stricter limits for sensitive operations.
* Combine per-user, per-client, and per-IP limits.
* Return clear retry information.
* Monitor rejection metrics.
* Use distributed storage for multi-instance deployments.
* Test rate limiting behavior under load.

---

## Common Mistakes

### No Rate Limiting

Every public API should have some form of protection.

### Overly Aggressive Limits

Legitimate users should not be penalized.

### Limiting Internal Events

Backend service-to-service communication often requires separate policies.

### Ignoring Monitoring

Rate limiting is only effective if operators can observe enforcement behavior.

---

## Design Principle

Rate limiting is not only a security feature.

It is a reliability mechanism that protects systems, preserves service availability, and ensures fair access to shared resources.

A well-designed platform should fail gracefully under load rather than collapse under uncontrolled traffic.
