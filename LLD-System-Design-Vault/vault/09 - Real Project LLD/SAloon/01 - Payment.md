---
tags: [saloon, real-project]
---

# SAloon — Payment Module

## 🧠 Problem
Support multiple payment methods (online via Razorpay/Stripe, and cash paid at the salon) — where only online payments can be auto-refunded — without `BookingService` needing special-case logic per method.

## 🤔 Why does the problem exist
The first draft had `CashPayment extends Payment` alongside `OnlinePayment extends Payment`, with `refund()` on the base class — `CashPayment.refund()` had to throw, which is a textbook [[../../02 - SOLID/02 - LSP|LSP]] violation I hit for real.

## 💡 Idea
Split `Payment` (the ability to pay) from `Refundable` (the ability to refund) as separate interfaces. Each payment gateway (`RazorpayAdapter`, `StripeAdapter`) implements both; `CashPayment` implements only `Payment`. Third-party SDKs are wrapped in [[../../05 - Design Patterns/Structural/Adapter|Adapters]].

## 🏗️ Structure
```
Payment (interface) — pay(amount)
Refundable (interface) — refund(amount)

RazorpayAdapter implements Payment, Refundable  → wraps RazorpayClient SDK
StripeAdapter implements Payment, Refundable    → wraps Stripe SDK
CashPayment implements Payment                  → no Refundable, honestly
```

## ☕ Java Implementation
```java
interface Payment { void pay(double amountInRupees); }
interface Refundable { void refund(double amountInRupees); }

class RazorpayAdapter implements Payment, Refundable {
    private final RazorpayClient client;
    public void pay(double amt) { client.initiateTxn(orderId, (int)(amt * 100)); }
    public void refund(double amt) { client.refund(orderId, (int)(amt * 100)); }
}

class CashPayment implements Payment {
    public void pay(double amt) { /* mark as pending collection at desk */ }
    // no refund() — correctly reflects reality, doesn't lie about capability
}
```

## Where Law of Demeter also showed up here
`BookingService` calls `customer.pay(amount)` — never `booking.getCustomer().getWallet().getBalance()...`. `Customer` hides how payment actually happens internally. See [[../../03 - Design Principles/03 - Law of Demeter]].

## Every pattern this module uses
- [[../../05 - Design Patterns/Structural/Adapter]] — wrapping Razorpay/Stripe SDKs
- [[../../02 - SOLID/02 - LSP]] — the Refundable split
- [[../../02 - SOLID/04 - DIP]] — `BookingService` depends on `Payment`, never a concrete gateway
- [[../../03 - Design Principles/03 - Law of Demeter]] — `customer.pay()`, not chained field access

## 🔥 Interview Question
*"Why not just have `refund()` throw `UnsupportedOperationException` for cash?"*
→ Because then any code calling `payment.refund()` generically has to know, out of band, which concrete type it's holding to avoid crashing — defeating polymorphism. Splitting the interface means the type system itself prevents calling refund on something that was never refundable.

## 🔗 Related
[[00 - Booking]]
[[../../02 - SOLID/02 - LSP]]

#saloon
