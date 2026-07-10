# Transactional Outbox Pattern

> A data management pattern that guarantees reliable event publication by storing business data and domain events in the same database transaction.

---

# Table of Contents

- Overview
- Problem
- Why Not Distributed Transactions (2PC)?
- Solution
- How It Works
- Architecture
- Outbox Table Design
- Event Lifecycle
- Polling Publisher
- Change Data Capture (CDC)
- Polling vs CDC
- Message Delivery Guarantees
- Failure Scenarios
- Advantages
- Disadvantages
- When to Use
- When NOT to Use
- Common Mistakes
- Best Practices
- Related Patterns
- Spring Boot Example
- Interview Questions
- References

---

# Overview

In a microservices architecture, a business operation often requires two independent actions:

1. Persist business data to the database.
2. Publish a domain event to a message broker (Kafka, RabbitMQ, etc.).

These two operations cannot be wrapped in a single ACID transaction.

The Transactional Outbox Pattern solves this problem by storing the domain event in an Outbox Table within the same database transaction as the business data.

A separate process later publishes these events to the message broker.

---

# Problem

Suppose Order Service creates a new order.

```
Create Order

↓

Save Order

↓

Publish OrderCreated Event
```

What happens if the database commit succeeds but Kafka is unavailable?

```
Save Order          ✅

Publish Event       ❌
```

Result:

- Order exists.
- No event is published.
- Payment Service never receives the event.
- Inventory Service never reserves stock.
- System becomes inconsistent.

---

# Why Not Distributed Transactions (2PC)?

Distributed transactions (XA / Two-Phase Commit) attempt to coordinate both the database and message broker in a single transaction.

However, they introduce:

- Tight coupling
- Reduced scalability
- High latency
- Complex recovery
- Limited support in cloud-native systems

Modern microservice architectures generally avoid XA transactions in favor of eventual consistency.

---

# Solution

Instead of publishing directly to Kafka:

```
BEGIN TRANSACTION

↓

Insert Order

↓

Insert Outbox Event

↓

COMMIT
```

Later:

```
Outbox Publisher

↓

Read Outbox Event

↓

Publish to Kafka

↓

Mark as Published
```

Since both the Order and Outbox Event are saved in the same transaction, they either succeed or fail together.

---

# How It Works

1. Client sends a request.
2. Order Service starts a database transaction.
3. Save business data.
4. Save Outbox Event.
5. Commit transaction.
6. Background Publisher reads pending events.
7. Publish event to Kafka.
8. Kafka acknowledges the message.
9. Update event status to PUBLISHED.

---

# Architecture

> Add architecture diagram here.

---

# Outbox Table Design

Example:

```sql
CREATE TABLE outbox_events (

    id UUID PRIMARY KEY,

    aggregate_type VARCHAR(100),

    aggregate_id VARCHAR(100),

    event_type VARCHAR(100),

    payload JSONB,

    status VARCHAR(20),

    created_at TIMESTAMP,

    published_at TIMESTAMP
);
```

### Column Description

| Column | Description |
|----------|-------------|
| id | Unique event identifier |
| aggregate_type | Aggregate type (Order, Payment, Customer...) |
| aggregate_id | Aggregate identifier |
| event_type | Event name |
| payload | Serialized event payload |
| status | Current publishing status |
| created_at | Event creation time |
| published_at | Publishing time |

---

# Event Lifecycle

```
PENDING

↓

PROCESSING

↓

PUBLISHED
```

or

```
PENDING

↓

PROCESSING

↓

FAILED

↓

Retry

↓

PROCESSING

↓

PUBLISHED
```

Status meanings:

- **PENDING** – Waiting to be published.
- **PROCESSING** – Currently being processed by a worker.
- **PUBLISHED** – Successfully published.
- **FAILED** – Publishing failed and will be retried.

---

# Polling Publisher

A scheduled worker periodically checks the Outbox Table.

```
Scheduler

↓

Read PENDING Events

↓

Mark PROCESSING

↓

Publish to Kafka

↓

Kafka ACK

↓

Mark PUBLISHED
```

Advantages:

- Easy to implement
- No additional infrastructure

Disadvantages:

- Periodic database polling
- Slight publishing delay

---

# Transaction Log Tailing

Instead of polling, Transaction Log Tailing monitors the database transaction log.

Example using Debezium(CDC Tool):

```
Order Service

↓

Orders Table

+

Outbox Table

↓

Database Transaction Log
(WAL / Binlog)

↓

Debezium

↓

Kafka
```

The application never publishes events.

Nobody polls the Outbox table.
Debezium continuously watches the database transaction log.

Advantages:

- Near real-time
- Lower database load
- No polling

Disadvantages:

- Additional infrastructure
- More operational complexity

---
So basically CDC in general is a tool that monitors the database transaction log and publishes events to Kafka.

```
                    Change Data Capture (CDC)
                             │
        ┌────────────────────┴────────────────────┐
        │                                         │
Polling Publisher                    Transaction Log Tailing
(Read Outbox Table)                  (Read DB WAL/Binlog)
```



# Polling vs Transaction Log Tailing

| Feature | Polling | Transaction Log Tailing |
|----------|----------|------|
| Complexity | Low | Medium |
| Infrastructure | Scheduler | Debezium |
| Database Load | Higher | Lower |
| Latency | Seconds | Near Real-Time |
| Recommended For | Small Projects | Enterprise Systems |

---

# Message Delivery Guarantees

Messaging systems generally provide three delivery guarantees.

## At Most Once

```
Send

↓

Lost

↓

Never Retried
```

Message may be lost.

---

## At Least Once

```
Send

↓

Retry

↓

Duplicate Possible
```

Messages are never intentionally lost, but duplicates may occur.

**Transactional Outbox provides this guarantee.**

---

## Exactly Once

Each message is processed exactly once.

This requires additional broker and consumer coordination and is outside the scope of the Outbox Pattern.

---

# Failure Scenarios

## Scenario 1

Without Outbox

```
Save Order      ✅

Publish Kafka   ❌
```

Result:

Lost event.

---

## Scenario 2

With Outbox

```
Save Order      ✅

Save Outbox     ✅

Commit          ✅

Crash
```

No problem.

The event remains in the Outbox Table.

It will be published after restart.

---

## Scenario 3

```
Publish Kafka

↓

Kafka ACK

↓

Application Crash

↓

Update Status
```

The event is already in Kafka, but the Outbox row is still marked as `PENDING`.

The worker republishes the event after restart.

Consumers must therefore be **idempotent**.

---

## Scenario 4

Kafka unavailable.

```
Publish

↓

FAILED

↓

Retry

↓

PUBLISHED
```

---

# Advantages

- Reliable event publication
- Atomic database transaction
- No distributed transactions
- Supports Event-Driven Architecture
- Supports Saga Pattern
- Better fault tolerance
- Technology independent

---

# Disadvantages

- Additional Outbox Table
- Additional Publisher Process
- Eventual consistency
- More infrastructure
- Requires cleanup strategy

---

# When to Use

✅ Kafka

✅ RabbitMQ

✅ Event-Driven Architecture

✅ Saga Pattern

✅ Database per Service

✅ Reliable event publishing

---

# When NOT to Use

❌ Monolithic CRUD applications

❌ Systems without messaging

❌ Applications where eventual consistency is unnecessary

---

# Common Mistakes

## Publishing Directly After Commit

```
Commit Database

↓

Publish Kafka
```

Events may be lost.

---

## Publishing Before Commit

```
Publish Kafka

↓

Rollback Database
```

Consumers receive events for data that doesn't exist.

---

## Using Boolean Published Flag

Instead of:

```
published = true
```

Prefer:

```
PENDING

PROCESSING

FAILED

PUBLISHED
```

---

## Ignoring Idempotency

Duplicate events can occur.

Consumers must safely process duplicate messages.

---

## No Cleanup Strategy

Outbox Tables continuously grow.

Archive or delete old published events.

---

# Best Practices

- Store Outbox events in the same transaction.
- Use immutable domain events.
- Make consumers idempotent.
- Implement retries with exponential backoff.
- Archive published events.
- Monitor failed publications.
- Add Correlation IDs.
- Use batch publishing.
- Use `SELECT ... FOR UPDATE SKIP LOCKED` (or equivalent) to safely claim events in multi-worker environments.
- Prefer CDC for high-throughput systems.

---

# Related Patterns

## Saga

Uses reliable event publication to coordinate distributed transactions.

---

## Change Data Capture (CDC)

Alternative implementation for publishing Outbox events.

---

## Event Sourcing

Stores domain events as the source of truth rather than just using events for integration.

---

## Database per Service

Each service owns its own Outbox Table.

---

## Idempotent Consumer

Required because duplicate event delivery is possible.

---

## Retry Pattern

Handles temporary broker failures.

---

## Dead Letter Queue

Stores permanently failed messages.

---

# Spring Boot Example

Repository structure:

```
springboot-microservices-examples/

patterns/

└── transactional-outbox/

    ├── polling-publisher/

    └── debezium-cdc/
```

Examples include:

- Spring Boot
- Spring Kafka
- PostgreSQL
- RabbitMQ (optional)
- Debezium
- Docker Compose

---

# Interview Questions

### What problem does the Transactional Outbox Pattern solve?

Reliable event publication without distributed transactions.

---

### Why can't we simply use `@Transactional` and `kafkaTemplate.send()`?

The database and Kafka participate in separate transactions.

One can succeed while the other fails.

---

### Why isn't XA Transaction recommended?

It reduces scalability, increases complexity, and is poorly suited for cloud-native architectures.

---

### What happens if the application crashes after Kafka acknowledges the message?

The Outbox event may still be marked as `PENDING`.

The publisher retries the event.

Consumers must therefore be idempotent.

---

### Why is idempotency required?

The Outbox Pattern guarantees **at-least-once delivery**, so duplicate messages are possible.

---

### How do you prevent two workers from publishing the same event?

Use optimistic claiming or database locking, such as:

- `SELECT ... FOR UPDATE SKIP LOCKED` (PostgreSQL)
- Atomic status updates (`PENDING → PROCESSING`)

---

### Polling or CDC?

Polling is simpler.

CDC provides better scalability and lower database load.

---

### Can Transactional Outbox work with RabbitMQ?

Yes.

It is independent of the message broker.

---

### Does the Outbox Pattern guarantee Exactly Once delivery?

No.

It guarantees **At-Least-Once** delivery.

Exactly-once processing requires additional broker and consumer mechanisms.
