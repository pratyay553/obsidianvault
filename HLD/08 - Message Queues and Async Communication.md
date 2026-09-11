---
tags: [hld]
---

# Message Queues & Async Communication

## 🧠 Problem
When a booking is confirmed, SAloon's `BookingService` directly calls the email service, the SMS service, and the analytics service — all synchronously, in the request path. If the SMS provider is slow today, every booking request hangs waiting for it, even though the customer doesn't actually need the SMS to have been *sent* before their booking confirmation page loads.

## 🤔 Why does the problem exist
Calling another service directly (synchronously) couples your request's speed and reliability to *that service's* speed and reliability — even when the work doesn't need to happen before you respond to the user. It also means if the SMS service is briefly down, bookings fail entirely, even though nothing about the booking itself was actually broken.

## 💡 Idea
A **message queue** decouples the producer of work from the consumer of work: the producer drops a message ("booking #123 confirmed") onto a queue and moves on immediately; one or more consumers pull messages off the queue and process them independently, at their own pace, with retries on failure.

## 🏗️ Structure
```
BookingService ──(publish)──▶ [ Queue ] ──(consume)──▶ EmailWorker
                                    │──────────────────▶ SmsWorker
                                    └──────────────────▶ AnalyticsWorker

BookingService returns to the client immediately after publishing —
it doesn't wait for Email/Sms/Analytics to finish.
```

## ⚙️ Key concepts
- **Producer / Consumer** — who publishes messages vs who processes them. Multiple consumers can read from the same queue for parallel processing.
- **At-least-once vs exactly-once delivery** — most queues guarantee a message is delivered at least once (it might be redelivered if a consumer crashes mid-processing), so consumers should be **idempotent** (processing the same message twice causes no harm).
- **Pub/Sub vs point-to-point** — point-to-point: one message, one consumer takes it. Pub/Sub: one message, *every* subscriber gets a copy (e.g. Kafka topics, SNS).
- **Backpressure** — if consumers fall behind, the queue absorbs the backlog instead of the producer failing; this is what protects `BookingService` from a slow SMS provider.

## 🏢 SAloon Example
`BookingService` publishes a single `BookingConfirmed` event to a queue and returns to the client in milliseconds. `EmailWorker`, `SmsWorker`, and `AnalyticsWorker` each independently consume that event on their own schedule. If the SMS provider goes down for ten minutes, SMS messages simply queue up and get sent once it recovers — bookings themselves never fail because of it. Note this is the async, cross-service sibling of [[00 - SRP]]: `BookingService` still only knows how to manage a booking; it doesn't know or care *how* a confirmation gets delivered.

## 🎯 When should I use it?
Any work that doesn't need to finish before you respond to the user (notifications, analytics, thumbnail generation, report emails), or work you want to survive a downstream service being temporarily unavailable.

## ⚠️ When NOT to use it?
Don't queue work the user is actively waiting on the result of — e.g. "is this slot available" during checkout needs a synchronous answer, not a queued one. Queues also add operational complexity (another moving piece to run and monitor) and eventual — not immediate — consistency, so don't reach for one just because it sounds scalable.

## 🔥 Interview Question
*"Why must consumers of a message queue be idempotent?"*
→ Most queues guarantee at-least-once delivery: if a consumer crashes after processing a message but before acknowledging it, the queue will redeliver that message. If processing it twice has a side effect (e.g. charging a card twice), that's a bug — so consumers must be written so that handling the same message multiple times produces the same end result as handling it once.

## 🔗 Related
[[00 - SRP]]
[[00 - Introduction to HLD]]
