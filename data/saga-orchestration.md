# Saga Pattern – Orchestration

> A centralized Saga implementation where a dedicated orchestrator coordinates the execution of local transactions across multiple microservices.

---

# Table of Contents

- Overview
- Problem
- Solution
- How It Works
- Architecture
- Workflow
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

Saga Orchestration is a centralized implementation of the Saga Pattern.

Instead of services reacting to each other's events, a dedicated Saga Orchestrator controls the entire business workflow.

The orchestrator decides:

- Which service to call next
- When to execute a compensating transaction
- When the Saga succeeds
- When the Saga fails

Each microservice only performs its own local transaction and reports the result back to the orchestrator.

---

# Problem

Business workflows often involve multiple services.

Example:

- Order Service
- Payment Service
- Inventory Service
- Shipping Service

If every service communicates with every other service, the workflow becomes difficult to understand and maintain.

```

Order

↓

Payment

↓

Inventory

↓

Shipping

```

As the workflow grows, event chains become increasingly complex.

---

# Solution

Introduce a central Saga Orchestrator.

```

                    Saga Orchestrator
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    Order Service   Payment Service   Inventory Service
                                              │
                                              ▼
                                      Shipping Service

```

The orchestrator keeps track of the Saga state and decides the next step.

---

# How It Works

## Successful Flow

1. Customer places an order.
2. Orchestrator calls Order Service.
3. Order Service creates the order.
4. Orchestrator calls Payment Service.
5. Payment succeeds.
6. Orchestrator calls Inventory Service.
7. Inventory reserves stock.
8. Orchestrator calls Shipping Service.
9. Shipping creates shipment.
10. Saga completed.

---

## Failure Flow

Suppose Inventory fails.

The orchestrator executes compensation.

```

Reserve Inventory 

↓

Refund Payment

↓

Cancel Order

↓

Saga Failed

```

The orchestrator controls every compensation step.

---

# Architecture

> Add architecture diagram here.

---

# Workflow

## Success

```

Customer

↓

Saga Orchestrator

↓

Order Service

↓

Payment Service

↓

Inventory Service

↓

Shipping Service

↓

Completed

```

---

## Compensation

```

Inventory Failed

↓

Saga Orchestrator

↓

Refund Payment

↓

Cancel Order

↓

Saga Failed

```

---

# Example

## Step 1

Orchestrator

↓

Create Order

↓

Order Service

---

## Step 2

Orchestrator

↓

Charge Payment

↓

Payment Service

---

## Step 3

Orchestrator

↓

Reserve Inventory

↓

Inventory Service

---

## Step 4

Orchestrator

↓

Create Shipment

↓

Shipping Service

---

# Advantages

- Easy to understand
- Centralized workflow
- Easier monitoring
- Easier debugging
- Better visibility
- Simpler compensation logic

---

# Disadvantages

- Additional service to maintain
- Single coordination point
- Orchestrator can become complex
- Workflow changes require orchestrator updates

---

# When to Use

✅ Complex business workflows

✅ Banking

✅ Insurance

✅ Travel booking

✅ Enterprise applications

✅ Long-running business processes

---

# When NOT to Use

❌ Very small workflows

❌ Simple CRUD applications

❌ Highly decoupled event-driven systems

---

# Common Mistakes

## Putting Business Logic Inside the Orchestrator

The orchestrator should coordinate the workflow.

Business rules belong inside the services.

---

## Creating a God Orchestrator

One orchestrator should not manage every business process.

Split orchestrators by business capability.

Example:

- Order Saga Orchestrator
- Payment Saga Orchestrator
- Booking Saga Orchestrator

---

## Long Synchronous Calls

Avoid blocking the orchestrator for long-running tasks.

Prefer asynchronous callbacks or events.

---

## Ignoring Timeouts

The orchestrator should detect:

- Service timeout
- Retry exhaustion
- Service unavailable

---

## Missing Compensation Logic

Every forward action should have a corresponding compensating action.

---

# Best Practices

- Keep the orchestrator lightweight.
- Store Saga state.
- Implement retries.
- Use Circuit Breakers.
- Use Correlation IDs.
- Make compensation idempotent.
- Persist Saga progress.
- Monitor Saga execution.
- Use Outbox Pattern when publishing events.

---

# Spring Boot Example

- Full implementation(Soon)

---

# Interview Questions

### What is Saga Orchestration?

A centralized Saga implementation where a dedicated orchestrator coordinates all local transactions.

---

### What is the biggest advantage?

Centralized workflow management and easier debugging.

---

### What is the biggest disadvantage?

The orchestrator can become a bottleneck if it grows too large.

---

### Can the orchestrator use Kafka?

Yes.

It can communicate using:

- REST
- gRPC
- Kafka
- RabbitMQ

The communication mechanism is independent of the orchestration pattern.

---

### Does the orchestrator execute business logic?

No.

It coordinates the workflow.

Each service owns its own business logic.

---

# Related Patterns

- Saga Pattern
- Saga Choreography
- Transactional Outbox
- Circuit Breaker
- Retry
- Idempotent Consumer
- Database per Service
