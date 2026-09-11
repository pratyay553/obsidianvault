---
tags: [pattern, behavioral]
---

# Chain of Responsibility

## 🧠 Problem
An incoming SAloon booking request needs several checks before it's allowed through: is the request authenticated, is the customer rate-limited, is the slot even valid. I don't want one giant method doing all three checks with nested `if`s.

## 🤔 Why does the problem exist
Nested validation `if`s inside one method get hard to reorder, hard to skip conditionally, and hard to test in isolation.

## 💡 Idea
Each check becomes its own handler. Handlers are linked into a chain; each either handles the request or passes it to the next.

## 🏗️ Structure
```
AuthHandler → RateLimitHandler → SlotValidityHandler → (actual booking logic)
```

## ☕ Java Implementation
```java
abstract class BookingHandler {
    protected BookingHandler next;
    void setNext(BookingHandler next) { this.next = next; }
    abstract void handle(BookingRequest req);
}

class AuthHandler extends BookingHandler {
    void handle(BookingRequest req) {
        if (!req.isAuthenticated()) throw new UnauthorizedException();
        if (next != null) next.handle(req);
    }
}
class RateLimitHandler extends BookingHandler {
    void handle(BookingRequest req) {
        if (limiter.isLimited(req.customerId())) throw new TooManyRequestsException();
        if (next != null) next.handle(req);
    }
}
```

## 🏢 SAloon Example
The request-validation pipeline in front of `BookingFacade` — see [[09 - Real Project LLD/SAloon/00 - Booking]] and [[08 - LLD Problems/09 - Rate Limiter]]. Reordering checks (e.g. rate-limit before auth to fail cheaply) is just reordering the chain, not rewriting logic.

## 🎯 When should I use it?
A request needs to pass through a variable, orderable set of checks/handlers, and you want each check isolated and independently testable.

## ⚠️ When NOT to use it?
If the checks are always exactly the same 2 steps and will never be reordered or extended, plain sequential `if`s in one method are more readable.

## 🔥 Interview Question
*"How would you make the chain order configurable without code changes?"*
→ Build the chain from an ordered config list at startup (`List<BookingHandler>` iterated instead of hardcoded `.setNext()` calls) — same pattern, more flexible wiring.

## 🔗 Related
[[../Structural/Proxy]]
[[08 - LLD Problems/09 - Rate Limiter]]

#pattern/behavioral
