---
tags: [lld-problem]
---

# Rate Limiter

## 🧠 Problem
Design a rate limiter that caps how many requests a customer can make (e.g. booking attempts) within a time window, to prevent abuse — without letting a burst of requests all sneak through right at a window boundary.

## 🤔 Why does the problem exist
Naive "count requests per fixed minute window" resets to zero at each minute boundary, so a client can send a full window's worth of requests at 0:59 and another full window's worth at 1:00 — double the intended rate right at the boundary.

## 💡 Idea
Encapsulate the actual algorithm (fixed window, sliding window, token bucket) behind a `RateLimitStrategy`, so it's swappable, and wrap the protected service behind a [[../05 - Design Patterns/Structural/Proxy|Proxy]] that checks the limiter before delegating.

## 🏗️ Structure
```
RateLimiter (interface) → FixedWindowLimiter, SlidingWindowLimiter, TokenBucketLimiter
RateLimitedServiceProxy — checks limiter, then delegates to the real service
```

## ☕ Java Implementation (token bucket)
```java
class TokenBucketLimiter implements RateLimiter {
    private final int capacity; private double tokens; private long lastRefillMs;
    private final double refillPerMs;

    synchronized boolean tryAcquire() {
        refill();
        if (tokens >= 1) { tokens -= 1; return true; }
        return false;
    }
    private void refill() {
        long now = System.currentTimeMillis();
        tokens = Math.min(capacity, tokens + (now - lastRefillMs) * refillPerMs);
        lastRefillMs = now;
    }
}
```

## 🏢 SAloon Tie-in
`RateLimitedBookingServiceProxy` — see [[../05 - Design Patterns/Structural/Proxy]] and [[../09 - Real Project LLD/SAloon/00 - Booking]], protecting the booking endpoint from bot spam.

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/Strategy]] — swappable limiting algorithm
- [[../05 - Design Patterns/Structural/Proxy]] — intercepts calls before the real service
- [[../05 - Design Patterns/Behavioral/Chain of Responsibility]] — combined with other request checks (auth, then rate limit)

## 🔥 Interview Question
*"Fixed window vs sliding window vs token bucket — when do you pick which?"*
→ Fixed window: simplest, but allows boundary bursts. Sliding window: smooths that out, more memory/compute. Token bucket: naturally allows short bursts up to the bucket size while enforcing a steady average rate — usually the best default for APIs.

## 🔗 Related
[[../05 - Design Patterns/Structural/Proxy]]
[[../05 - Design Patterns/Behavioral/Chain of Responsibility]]

#lld-problem
