# Saga Pattern

> A distributed transaction pattern that maintains data consistency across multiple microservices by coordinating a sequence of local transactions.

---

# Table of Contents

- Overview
- Problem
- Solution
- Why Do We Need It?
- Architecture
- Types of Saga
- How It Works
- Example
- Advantages
- Disadvantages
- When to Use
- When NOT to Use
- Common Mistakes
- Best Practices
- Related Patterns
- Spring Boot Example

---

# Overview

The Saga Pattern is a distributed transaction pattern used to maintain data consistency across multiple microservices.

Instead of using a single ACID transaction spanning multiple services, a Saga breaks the business transaction into a sequence of local transactions.

Each service updates its own database and then either:

- Publishes an event (Choreography)
- Invokes the next service through a central orchestrator (Orchestration)

If one step fails, previously completed steps execute compensating transactions to undo their work.

---

# Problem

In a monolithic application, a single database transaction guarantees consistency.

```

Order

↓

Payment

↓

Inventory

↓

Shipping

(All inside one ACID transaction)

```

In a microservices architecture, every service owns its own database.

```

Order Service
↓

OrderDB

Payment Service
↓

PaymentDB

Inventory Service
↓

InventoryDB

```

Since there is no distributed ACID transaction across services, partial failures can leave the system in an inconsistent state.

Example:

- Order created 
- Payment completed 
- Inventory reservation failed 

Without compensation, the customer has already paid but no inventory is reserved.

---

# Solution

A Saga coordinates multiple local transactions.

If every step succeeds:

```

Order
↓

Payment
↓

Inventory
↓

Shipping

```

If any step fails:

```

Shipping ❌

↑

Release Inventory

↑

Refund Payment

↑

Cancel Order

```

Each completed transaction is compensated in reverse order.

---

# Why Do We Need It?

The Saga Pattern provides:

- Distributed transaction management
- Eventual consistency
- Service autonomy
- Independent databases
- Failure recovery
- Better scalability

---

# Architecture

> Add architecture diagram here.

---

# Types of Saga

## Choreography

Services communicate using events.

```

Order
↓

OrderCreated Event

↓

Payment

↓

PaymentCompleted Event

↓

Inventory

↓

InventoryReserved Event

↓

Shipping

```

No central coordinator exists.

---

## Orchestration

A central orchestrator controls the workflow.

```

Saga Orchestrator

↓

Order

↓

Payment

↓

Inventory

↓

Shipping

```

The orchestrator decides what happens next.

---

# How It Works

Example:

1. Customer places an order.
2. Order Service saves the order.
3. Order Service publishes `OrderCreated`.
4. Payment Service charges the customer.
5. Payment Service publishes `PaymentCompleted`.
6. Inventory Service reserves stock.
7. Shipping Service creates the shipment.

If Inventory fails:

1. Inventory publishes `InventoryFailed`.
2. Payment refunds the customer.
3. Order Service cancels the order.

---

# Example

## Successful Flow

```

Order Created

↓

Payment Success

↓

Inventory Reserved

↓

Shipment Created

```

---

## Failure Flow

```

Order Created

↓

Payment Success

↓

Inventory Failed

↓

Refund Payment

↓

Cancel Order

```

---

# Advantages

- No distributed transactions
- Independent services
- Better scalability
- Fault tolerance
- Eventual consistency
- Supports asynchronous communication

---

# Disadvantages

- Complex implementation
- Compensation logic required
- Difficult debugging
- Eventual consistency instead of immediate consistency
- More infrastructure

---

# When to Use

✅ Database per Service

✅ Event-driven architecture

✅ Independent microservices

✅ Long-running business processes

---

# When NOT to Use

❌ Monolithic applications

❌ Simple CRUD systems

❌ Single database applications

❌ Business operations requiring immediate global ACID consistency

---

# Common Mistakes

### 1. Expecting ACID Transactions

Saga provides eventual consistency, not distributed ACID.

---

### 2. Forgetting Compensation

Every local transaction should have a corresponding compensating transaction.

---

### 3. Long-Running Sagas

Very long workflows increase the likelihood of failures.

---

### 4. Ignoring Idempotency

Compensation and event handlers should be idempotent.

---

### 5. Mixing Choreography and Orchestration Without Clear Boundaries

Choose the appropriate approach based on workflow complexity.

---

# Best Practices

- Keep local transactions small.
- Design compensating transactions first.
- Make event handlers idempotent.
- Use Transactional Outbox to publish events reliably.
- Include Correlation IDs for tracing.
- Monitor Saga execution.

---

# Related Patterns

- Database per Service
- Transactional Outbox
- CQRS
- Event Sourcing
- Change Data Capture (CDC)
- Circuit Breaker
- Retry

---

# Spring Boot Example

- Choreography(Soon)

- Orchestration(Soon)