# Saga Pattern – Choreography

> A decentralized Saga implementation where each microservice reacts to domain events and decides what to do next without a central coordinator.

---

# Table of Contents

- Overview
- Problem
- Solution
- How It Works
- Architecture
- Event Flow
- Example
- Advantages
- Disadvantages
- When to Use
- When NOT to Use
- Common Mistakes
- Best Practices
- Spring Boot Example
- Interview Questions

---

# Overview

Saga Choreography is a decentralized implementation of the Saga Pattern.

Instead of having a central orchestrator controlling the business workflow, each service publishes domain events after completing its local transaction.

Other services subscribe to these events and execute their own local transactions.

Every service knows:

- Which events to consume
- Which events to publish

No service knows the entire business workflow.

---

# Problem

Suppose an e-commerce application contains:

- Order Service
- Payment Service
- Inventory Service
- Shipping Service

Each service owns its own database.

A customer places an order.

The business transaction spans multiple services.

Traditional distributed transactions (2PC/XA) are:

- Slow
- Difficult to scale
- Reduce service autonomy

---

# Solution

Each service performs:

1. Local transaction
2. Publish an event

The next service reacts to the event.

```

Order Service
↓

OrderCreated Event

↓

Payment Service

↓

PaymentCompleted Event

↓

Inventory Service

↓

InventoryReserved Event

↓

Shipping Service

```

If a step fails, another event starts the compensation process.

---

# How It Works

## Successful Flow

1. Customer places an order.
2. Order Service stores the order.
3. Order Service publishes **OrderCreated**.
4. Payment Service receives the event.
5. Payment Service charges the customer.
6. Payment Service publishes **PaymentCompleted**.
7. Inventory Service reserves stock.
8. Inventory Service publishes **InventoryReserved**.
9. Shipping Service creates the shipment.
10. Shipping Service publishes **ShipmentCreated**.

Business transaction completed.

---

## Failure Flow

Suppose Inventory fails.

```

OrderCreated

↓

PaymentCompleted

↓

InventoryFailed

↓

PaymentRefundRequested

↓

OrderCancelled

```

Each service performs its own compensating transaction.

---

# Architecture

> Add architecture diagram here.

---

# Event Flow

## Success

```

Customer

↓

Order Service

↓

OrderCreated

↓

Payment Service

↓

PaymentCompleted

↓

Inventory Service

↓

InventoryReserved

↓

Shipping Service

↓

ShipmentCreated

```

---

## Compensation

```

InventoryFailed

↓

RefundPayment

↓

PaymentRefunded

↓

CancelOrder

↓

OrderCancelled

```

---

# Example

## Order Service

Publishes:

```

OrderCreated

```

Consumes:

```

OrderCancelled

```

---

## Payment Service

Consumes:

```

OrderCreated

```

Publishes:

```

PaymentCompleted

PaymentFailed

PaymentRefunded

```

---

## Inventory Service

Consumes:

```

PaymentCompleted

```

Publishes:

```

InventoryReserved

InventoryFailed

```

---

## Shipping Service

Consumes:

```

InventoryReserved

```

Publishes:

```

ShipmentCreated

ShipmentFailed

```

---

# Advantages

- No central coordinator
- Services remain autonomous
- Loosely coupled
- Easy to add new subscribers
- Excellent scalability
- Fully event-driven

---

# Disadvantages

- Difficult to understand large workflows
- Event chains become long
- Harder debugging
- Complex monitoring
- Risk of cyclic dependencies
- Difficult to know the overall business state

---

# When to Use

✅ Event-driven architecture

✅ Kafka

✅ RabbitMQ

✅ Independent teams

✅ High scalability

✅ Loose coupling

---

# When NOT to Use

❌ Complex workflows involving many services

❌ Strict workflow control

❌ Business processes requiring centralized visibility

---

# Common Mistakes

## Using Events as Remote Procedure Calls

Events should represent something that has already happened.

Good:

```

OrderCreated

```

Bad:

```

CreateOrder

```

---

## Ignoring Idempotency

Consumers may receive duplicate events.

Handlers should be idempotent.

---

## Missing Correlation IDs

Without a Correlation ID, tracing a Saga becomes extremely difficult.

Always include:

- Saga ID
- Correlation ID
- Trace ID

---

## Not Using the Outbox Pattern

Publishing an event after committing the database transaction can result in lost events.

Use the Transactional Outbox Pattern.

---

## Circular Event Dependencies

Avoid workflows such as:

```

A → B → C → A

```

These create infinite loops and tightly coupled services.

---

# Best Practices

- Publish immutable domain events.
- Design idempotent consumers.
- Use Transactional Outbox.
- Include Correlation IDs.
- Implement retries.
- Use Dead Letter Topics (DLT).
- Monitor event processing.
- Keep events coarse-grained.
- Version your events.

---

# Spring Boot Example

Full implementation(Soon)
---

# Interview Questions

### What is Saga Choreography?

A decentralized Saga implementation where services communicate through events without a central orchestrator.

---

### What is the biggest advantage?

Loose coupling and high scalability.

---

### What is the biggest disadvantage?

As the number of services grows, understanding and debugging the workflow becomes difficult.

---

### Which messaging systems are commonly used?

- Apache Kafka
- RabbitMQ
- Amazon SQS/SNS
- Azure Service Bus

---

### Why is the Outbox Pattern commonly used with Choreography?

To guarantee reliable event publication after the local transaction commits.

---

### Why is idempotency important?

Message brokers may deliver duplicate messages.

Consumers must safely process duplicate events.

