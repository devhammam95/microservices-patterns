# Sidecar Pattern

> A deployment pattern where a helper component runs alongside a microservice in the same host or Kubernetes Pod, providing infrastructure capabilities without modifying the application code.

---

# Table of Contents

- Overview
- Problem
- Solution
- Why Do We Need It?
- How It Works
- Architecture
- Common Sidecar Responsibilities
- Real-World Examples
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

In a microservices architecture, applications often need infrastructure-related features such as:

- Logging
- Monitoring
- Security
- Service Discovery
- Traffic Management
- Configuration
- Secret Management

Instead of implementing these concerns inside every microservice, the **Sidecar Pattern** places a helper component alongside the application.

Both the application and the sidecar run together and communicate locally.

---

# Problem

Imagine every microservice has to implement:

- Logging
- TLS
- Metrics
- Configuration
- Service Registration

```
Order Service
├── Business Logic
├── Logging
├── Metrics
├── Security
├── Configuration
└── Service Discovery
```

Problems:

- Code duplication
- Difficult maintenance
- Inconsistent implementations
- Tight coupling between business and infrastructure logic

---

# Solution

Move infrastructure responsibilities into a Sidecar.

```
+--------------------------------------+
|              Pod / Host              |
|                                      |
|  +-------------------------------+   |
|  | Order Service                 |   |
|  | Business Logic Only           |   |
|  +---------------+---------------+   |
|                  |                   |
|  localhost       |                   |
|                  ▼                   |
|  +-------------------------------+   |
|  | Sidecar                       |   |
|  | Logging                       |   |
|  | Metrics                       |   |
|  | Proxy                         |   |
|  | Security                      |   |
|  +-------------------------------+   |
+--------------------------------------+
```

The application focuses only on business logic.

The sidecar handles cross-cutting concerns.

---

# Why Do We Need It?

The Sidecar Pattern provides:

- Separation of concerns
- Cleaner application code
- Reusable infrastructure
- Easier maintenance
- Independent upgrades
- Technology independence

---

# How It Works

1. The application starts.
2. The sidecar starts beside it.
3. Both communicate through localhost.
4. The sidecar handles infrastructure-related operations.
5. Business logic remains unchanged.

---

# Architecture

![Sidecar Architecture](../diagrams/deployment/sidecar-diagram.png.png)

---

# Common Sidecar Responsibilities

## Logging

Collect application logs and forward them to a central logging platform.

Example:

- Fluent Bit
- Fluentd
- Filebeat

---

## Metrics

Collect metrics and expose them to monitoring systems.

Example:

- Prometheus Sidecar

---

## Proxy

Intercept incoming and outgoing traffic.

Example:

- Envoy Proxy

Used for:

- mTLS
- Load Balancing
- Traffic Routing
- Retries
- Circuit Breakers

---

## Configuration

Retrieve configuration from a centralized configuration service.

---

## Secret Management

Retrieve secrets securely.

Example:

- HashiCorp Vault Agent

---

## Service Discovery

Register services automatically.

---

# Real-World Examples

| Sidecar | Purpose |
|----------|---------|
| Envoy | Service Proxy |
| Filebeat | Log Shipping |
| Vault Agent | Secret Management |

---

# Advantages

- Separation of business and infrastructure logic
- Reusable infrastructure components
- Easier maintenance
- Independent deployment and upgrades
- Consistent implementation across services
- Polyglot-friendly architecture

---

# Disadvantages

- More containers/processes
- Additional resource consumption
- Increased operational complexity
- More network hops (even if local)

---

# When to Use

✅ Kubernetes environments

✅ Service Mesh

✅ Centralized logging

✅ Distributed tracing

✅ Secret management

✅ Configuration management

✅ Traffic control

---

# When NOT to Use

❌ Small monolithic applications

❌ Very simple microservice architectures

❌ Infrastructure concerns already handled externally

---

# Common Mistakes

## Putting Business Logic in the Sidecar

The sidecar should only handle infrastructure concerns.

Business logic belongs inside the microservice.

---

## One Sidecar for Multiple Services

A sidecar should be dedicated to one service instance.

Sharing one sidecar across multiple services defeats the pattern.

---

## Tight Coupling

The application should continue functioning independently whenever possible.

Avoid creating strong runtime dependencies.

---

## Overusing Sidecars

Not every infrastructure concern requires a sidecar.

Sometimes a shared service or library is more appropriate.

---

# Best Practices

- Keep the sidecar focused on infrastructure.
- Communicate via localhost.
- Deploy the application and sidecar together.
- Make the sidecar independently upgradeable.
- Monitor both the application and the sidecar.
- Avoid putting domain logic in the sidecar.

---

# Related Patterns

- Ambassador Pattern
- Adapter Pattern
- API Gateway
- Service Mesh
- Circuit Breaker
- Retry

---

# Spring Boot Example
(Soon)

---

# Interview Questions

### What is the Sidecar Pattern?

A deployment pattern where an application runs alongside a helper component that provides infrastructure capabilities such as logging, security, or traffic management.

---

### Why use a Sidecar?

To separate business logic from infrastructure concerns and enable reusable, independently managed infrastructure components.

---

### Is the Sidecar deployed separately?

No.

The application and its sidecar are deployed together on the same host or Kubernetes Pod.

---

### How do they communicate?

Typically through **localhost** (loopback interface), making communication fast and secure.

---

### What are common Sidecar implementations?

- Vault Agent

---

### Is Sidecar the same as Ambassador?

No.

- **Sidecar** is a general deployment pattern for adding infrastructure capabilities.
- **Ambassador** is a specialized Sidecar that acts as a proxy for outbound communication.
