---
tags: [oop]
---

# Abstraction

## 🧠 Problem
My `PricingService` needs to calculate a price, but I don't want it to care whether pricing comes from a database, a config file, or a pricing microservice.

## 🤔 Why does the problem exist
If `PricingService` directly calls `MySQLPricingRepo.getRate()`, then it is now *married* to MySQL. Swap the DB, or add caching, or mock it in a test — and I have to touch `PricingService` itself. The "what" (get a rate) is now tangled with the "how" (SQL query).

## 💡 Idea
Expose only *what* an object does (a contract/interface). Hide *how* it does it behind that contract. Callers code against the interface, never the implementation.

## 🏗️ Structure
```
Client → PricingRepository (interface)
              ↑ implements
         MySQLPricingRepository
```

## ☕ Java Implementation
```java
interface PricingRepository {
    double getBaseRate(String serviceId);
}

class MySQLPricingRepository implements PricingRepository {
    public double getBaseRate(String serviceId) { /* SQL */ return 0; }
}

class PricingService {
    private final PricingRepository repo; // depends on the interface, not MySQL
    PricingService(PricingRepository repo) { this.repo = repo; }
}
```

## 🏢 SAloon Example
`PaymentGateway` interface abstracts Razorpay/Stripe. `PricingService` in [[09 - Real Project LLD/SAloon/04 - Pricing]] never knows which gateway processed the payment.

## 🎯 When should I use it?
Any time a class talks to something that could plausibly change: a DB, a third-party API, a notification channel, a pricing rule.

## ⚠️ When NOT to use it?
Don't add an interface for a class that will only ever have one implementation and no test-double need — that's abstraction for its own sake (violates [[03 - Design Principles/02 - YAGNI]]).

## 🔥 Interview Question
*"Interface vs abstract class — when do you pick which?"*
→ Interface: unrelated classes need to guarantee the same behavior, no shared state. Abstract class: subclasses share actual code/state.

## 🔗 Related
[[00 - Encapsulation]]
[[02 - SOLID/04 - DIP]]
[[05 - Design Patterns/Creational/Factory]]

#oop
