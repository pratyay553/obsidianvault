---
tags: [hld]
---

# API Gateway, Rate Limiting & Microservices vs Monolith

## 🧠 Problem
SAloon has grown into separate services — `BookingService`, `PaymentService`, `NotificationService` — each with its own URL. Mobile clients now need to know all three addresses, handle auth for each separately, and nothing stops a single misbehaving client from hammering `PaymentService` with 10,000 requests/sec.

## 🤔 Why does the problem exist
Splitting a system into independently-deployable services ([[00 - SRP]] applied at the architecture level) solves the problem of *one team's change breaking another team's service* — but it creates a new problem: clients now face many endpoints instead of one, and each service needs its own auth checks, its own protection from abuse, and its own cross-cutting logic (logging, rate limits) duplicated everywhere.

## 💡 Idea
An **API Gateway** is a single entry point that sits in front of all backend services: it routes requests to the right service, and centralizes cross-cutting concerns — authentication, rate limiting, request logging — so individual services don't each reinvent them. **Rate limiting** specifically caps how many requests a client can make in a time window, protecting services from abuse or accidental overload.

## 🏗️ Structure
```
Client ──▶ API Gateway ──▶ auth check, rate limit check
                 ├──▶ /bookings/* ──▶ BookingService
                 ├──▶ /payments/* ──▶ PaymentService
                 └──▶ /notify/*   ──▶ NotificationService
```

## ⚙️ Rate limiting algorithms
- **Fixed window:** count requests per client in fixed time slices (e.g. per minute); simple but allows bursts right at window boundaries.
- **Sliding window:** smooths that out by considering a rolling time window rather than fixed slices.
- **Token bucket:** each client has a bucket that refills with tokens at a fixed rate; each request consumes a token, requests are rejected when the bucket is empty — allows brief bursts up to the bucket size while capping sustained rate.

## ⚙️ Microservices vs Monolith
| | Monolith | Microservices |
|---|---|---|
| Deployment | One unit, deployed together | Each service deployed independently |
| Failure isolation | One bug can crash everything | One service crashing doesn't take down others |
| Team scaling | Hard for many teams to work without stepping on each other | Teams own separate services independently |
| Operational cost | Low — one thing to run | High — service discovery, network calls, distributed tracing, an API gateway |
| Data | One shared DB, easy joins/transactions | Each service ideally owns its own data — cross-service joins/transactions are hard |

## 🏢 SAloon Example
SAloon started as a monolith (fine for one team, low traffic). As booking volume and team size grew, `PaymentService` was split out first — it has the strictest reliability/compliance needs and the least reason to change alongside booking logic (a direct application of [[00 - SRP]] at the service level). An API Gateway now handles auth once for all three services, and applies a token-bucket rate limit per API key so one client can't starve `PaymentService` for everyone else.

## 🎯 When should I use it?
API Gateway: as soon as you have more than one backend service that clients talk to. Rate limiting: any public or partner-facing API. Microservices: once a monolith's team-coordination cost or differing scaling/reliability needs across features genuinely outweigh the operational overhead of running many services — not just because "microservices are the modern way."

## ⚠️ When NOT to use it?
Don't split into microservices before a monolith has actually demonstrated the pain (team collisions, one feature's load drowning another) that microservices solve — it's a real architectural cost, and a poorly-decomposed set of services is often worse than a well-organized monolith. This mirrors [[01 - OCP]]'s warning against speculative generality, just at a bigger scale.

## 🔥 Interview Question
*"What's the single biggest new problem microservices introduce that a monolith doesn't have?"*
→ Distributed data consistency: a monolith can wrap a multi-step change in one DB transaction; microservices, each owning their own data, generally can't — a booking-and-payment flow spanning two services needs patterns like sagas or eventual consistency ([[06 - CAP Theorem and Consistency Patterns]]) instead of a simple transaction.

## 🔗 Related
[[00 - SRP]]
[[08 - Message Queues and Async Communication]]
[[06 - CAP Theorem and Consistency Patterns]]
