---
tags: [pattern, structural]
---

# Proxy

## 🧠 Problem
Every SAloon booking action should first check "is this user rate-limited / authorized," but I don't want to sprinkle that check inside every method of `BookingService`.

## 🤔 Why does the problem exist
Cross-cutting checks (auth, rate limiting, caching) don't belong to the *core logic* of a class, but if not centralized, they get copy-pasted into every method that needs them.

## 💡 Idea
Create a class with the *same interface* as the real object, but that intercepts calls to add a check before delegating.

## 🏗️ Structure
```
BookingService (interface)
   ├── RealBookingService     (actual logic)
   └── RateLimitedBookingServiceProxy   (checks limit, then delegates)
```

## ☕ Java Implementation
```java
interface BookingService { Booking book(BookingRequest req); }

class RealBookingService implements BookingService {
    public Booking book(BookingRequest req) { return null; }
}

class RateLimitedBookingServiceProxy implements BookingService {
    private final BookingService real;
    private final RateLimiter limiter;
    public Booking book(BookingRequest req) {
        if (!limiter.tryAcquire(req.customerId())) throw new TooManyRequestsException();
        return real.book(req);
    }
}
```

## 🏢 SAloon Example
`RateLimitedBookingServiceProxy` wraps the real booking service at the API layer — see [[08 - LLD Problems/09 - Rate Limiter]] for the limiter itself, and [[09 - Real Project LLD/SAloon/00 - Booking]] for where it's wired in.

## 🎯 When should I use it?
Access control, rate limiting, lazy-loading expensive objects, caching results, logging around a real object.

## ⚠️ When NOT to use it?
If the "extra behavior" isn't about *controlling access* but genuinely new capability to every call, that's [[Decorator]] instead.

## 🔥 Interview Question
*"Proxy vs Decorator — the code literally looks the same, what's the difference?"*
→ Intent. Proxy controls *access* (can refuse to delegate at all). Decorator always delegates, but *adds* behavior on top.

## 🔗 Related
[[Decorator]]
[[08 - LLD Problems/09 - Rate Limiter]]

#pattern/structural
