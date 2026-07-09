# API Composition Pattern

> A communication pattern that composes data from multiple microservices into a single response for the client.

---

# Table of Contents

- Overview
- Problem
- Solution
- Why Do We Need It?
- Architecture
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
- Interview Questions


---

# Overview

API Composition is a communication pattern where an application gathers data from multiple microservices and combines it into a single response.

Instead of forcing clients to call several services individually, a composer (often an API Gateway or Backend for Frontend) retrieves the required data and returns one aggregated response.

---

# Problem

A client needs complete order details.

The required information is distributed across multiple services.

- Order Service
- Customer Service
- Payment Service
- Inventory Service

Without API Composition, the client must make multiple network calls.

```
Client
   │
   ├────────► Order Service
   ├────────► Customer Service
   ├────────► Payment Service
   └────────► Inventory Service
```

Problems:

- Increased latency
- More network traffic
- Complex client logic
- Multiple authentication requests
- Tight coupling between clients and services

---

# Solution

Introduce an API Composer.

```
Client
   │
   ▼
API Composer
   │
   ├────────► Order Service
   ├────────► Customer Service
   ├────────► Payment Service
   └────────► Inventory Service
```

The composer aggregates responses into one object.

---

# Why Do We Need It?

API Composition provides:

- Single client request
- Aggregated response
- Reduced network latency
- Simplified client implementation
- Better user experience
- Centralized aggregation logic

---

# Architecture

![API Composition Architecture](../diagrams/communication/api-composition-diagram.png)

---

# How It Works

Example:

1. Client requests `/orders/100`.
2. API Composer calls Order Service.
3. API Composer calls Customer Service.
4. API Composer calls Payment Service.
5. API Composer calls Inventory Service.
6. Responses are combined.
7. Client receives one JSON response.

---

# Example

Without API Composition

```
GET /orders/100

↓

Client

↓

GET Order

↓

GET Customer

↓

GET Payment

↓

GET Inventory
```

Four HTTP requests.

---

With API Composition

```
GET /orders/100

↓

API Composer

↓

Order Service

↓

Customer Service

↓

Payment Service

↓

Inventory Service

↓

Combined Response
```

One client request.

---

# Advantages

- Simplifies clients
- Reduces client network calls
- Aggregates distributed data
- Centralizes response composition
- Improves user experience

---

# Disadvantages

- Additional network hop
- Higher backend complexity
- Increased response latency
- Risk of cascading failures
- Difficult error handling

---

# When to Use

✅ Dashboard APIs

✅ Mobile applications

✅ Order summary pages

✅ Customer profile pages

✅ Product details pages

---

# When NOT to Use

❌ Simple CRUD APIs

❌ When one service already owns all required data

❌ High-throughput low-latency endpoints

---

# Common Mistakes

## Calling Too Many Services

Avoid composing data from dozens of services.

---

## Long Synchronous Chains

Too many synchronous requests increase latency.

---

## Ignoring Timeouts

Each downstream request should have a timeout.

---

## No Circuit Breaker

One unavailable service should not bring down the entire request.

---

## Business Logic in the Composer

The composer should aggregate data, not implement business rules.

---

# Best Practices

- Keep aggregation lightweight.
- Execute downstream requests in parallel.
- Use Circuit Breakers.
- Apply timeouts.
- Cache frequently requested data.
- Add Correlation IDs.
- Monitor downstream latency.
- Return partial responses when appropriate.

---

# Related Patterns

- API Gateway
- Backend for Frontend (BFF)
- CQRS
- Database per Service
- Cache Aside
- Circuit Breaker
- Retry

---

# Spring Boot Example

- Soon


---

# Interview Questions

### What problem does API Composition solve?

It aggregates data from multiple microservices into a single response.

---

### Is API Composition the same as API Gateway?

No.

API Gateway focuses on routing and cross-cutting concerns.

API Composition focuses on aggregating data from multiple services.

---

### Can API Gateway perform API Composition?

Yes.

Many API Gateways implement API Composition.

However, they are different patterns with different responsibilities.

---

### How can response latency be reduced?

- Execute calls in parallel.
- Cache responses.
- Apply timeouts.
- Use asynchronous requests.

---

### What are common technologies?

- Spring Cloud Gateway
- GraphQL
- gRPC
- REST
