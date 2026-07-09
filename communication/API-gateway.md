# API Gateway Pattern

> A communication pattern that provides a single entry point for client requests and routes them to the appropriate microservices.

---

# Table of Contents

- Overview
- Problem
- Solution
- Why Do We Need It?
- Architecture
- How It Works
- Example
- Responsibilities
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

The API Gateway Pattern introduces a single entry point between clients and backend microservices.

Instead of clients communicating directly with multiple services, every request first goes through the API Gateway.

The gateway is responsible for routing requests, enforcing security, and handling cross-cutting concerns.

---

# Problem

Without an API Gateway, clients must know every backend service.

```
                +----------------+
                |    Client      |
                +----------------+
                  /     |      \
                 /      |       \
                ▼       ▼        ▼
        Order Service  Payment Service  Inventory Service
```

Problems:

- Clients must know service locations.
- Multiple network requests.
- Tight coupling between clients and services.
- Authentication duplicated across services.
- Difficult API versioning.

---

# Solution

Introduce an API Gateway.

```
                 +----------------+
                 |    Client      |
                 +----------------+
                         |
                         ▼
                 +----------------+
                 |  API Gateway   |
                 +----------------+
                  /      |       \
                 ▼       ▼        ▼
        Order Service  Payment Service  Inventory Service
```

The gateway becomes the single entry point for all client requests.

---

# Why Do We Need It?

The API Gateway provides:

- Single entry point
- Request routing
- Authentication
- Authorization
- SSL termination
- Rate limiting
- Load balancing
- Request aggregation
- Response transformation
- API versioning

---

# Architecture

(Soon)

---

# How It Works

Example:

1. Client requests order details.
2. Request reaches API Gateway.
3. Gateway authenticates the user.
4. Gateway routes the request to Order Service.
5. Order Service returns the response.
6. Gateway sends the response back to the client.

For aggregated endpoints:

1. Gateway requests Order Service.
2. Gateway requests Payment Service.
3. Gateway requests Inventory Service.
4. Gateway combines responses.
5. Client receives a single response.

---

# Example

Without Gateway

```
Mobile App

↓

Order Service

↓

Payment Service

↓

Inventory Service

↓

Shipping Service
```

Four HTTP calls.

---

With Gateway

```
Mobile App

↓

API Gateway

↓

Order Service

↓

Payment Service

↓

Inventory Service

↓

Shipping Service
```

One client endpoint.

---

# Responsibilities

Typical gateway responsibilities include:

- Request Routing
- Authentication
- Authorization
- JWT Validation
- OAuth2 Integration
- Rate Limiting
- Request Aggregation
- Response Transformation
- Logging
- Monitoring
- CORS
- SSL Termination
- API Versioning
- Request Validation

---

# Advantages

- Single entry point
- Simplified clients
- Better security
- Centralized authentication
- Reduced client complexity
- Easier API evolution
- Cross-cutting concerns handled centrally

---

# Disadvantages

- Additional network hop
- Potential bottleneck
- Single point of failure (if not highly available)
- More infrastructure to manage

---

# When to Use

✅ Mobile applications

✅ Web applications

✅ Public APIs

✅ BFF architecture

✅ Large microservice systems

---

# When NOT to Use

❌ Small monolithic applications

❌ Internal systems with only one backend service

❌ Very simple architectures

---

# Common Mistakes

## 1. Putting Business Logic Inside the Gateway

The gateway should route requests.

Business logic belongs inside services.

---

## 2. Making the Gateway a Monolith

Avoid adding every responsibility to one huge gateway.

---

## 3. Direct Database Access

The gateway should never access databases.

Always communicate with services.

---

## 4. Calling Every Service Synchronously

Too many synchronous calls increase latency.

Use aggregation carefully.

---

## 5. Missing Rate Limiting

Public APIs should protect backend services from abuse.

---

# Best Practices

- Keep the gateway lightweight.
- Centralize authentication.
- Validate JWT tokens.
- Use HTTPS everywhere.
- Enable rate limiting.
- Implement request tracing.
- Add Correlation IDs.
- Keep business logic in services.
- Scale the gateway horizontally.

---

# Related Patterns

- Backend for Frontend (BFF)
- Service Discovery
- Circuit Breaker
- Retry
- Bulkhead
- Rate Limiting
- OAuth2
- JWT

---

# Spring Boot Example

Spring Cloud Gateway(soon)

---

# Interview Questions

### What problem does the API Gateway solve?

It provides a single entry point for clients while centralizing routing, authentication, and other cross-cutting concerns.

---

### Should business logic be placed in the gateway?

No.

Business logic belongs in backend services.

---

### Can an API Gateway aggregate responses?

Yes.

It can call multiple services and return a single combined response.

---

### Is an API Gateway required in every microservices architecture?

No.

It's recommended for most client-facing systems but may be unnecessary for very small internal architectures.

---

### What's the difference between an API Gateway and a Load Balancer?

A Load Balancer distributes traffic across service instances.

An API Gateway understands APIs and performs routing, authentication, rate limiting, request transformation, and other application-level functions.
