---
tags: [pattern, structural]
---

# Adapter

## 🧠 Problem
SAloon integrates Razorpay for payments, but their SDK exposes `RazorpayClient.initiateTxn(orderId, amountInPaise)` — completely different shape from the `PaymentGateway.pay(Booking, double)` interface the rest of the app expects.

## 🤔 Why does the problem exist
I can't modify Razorpay's SDK, and I don't want `BookingService` to know Razorpay's specific method names/units — that would leak a third-party detail into core business logic.

## 💡 Idea
Write a thin wrapper class that implements *my* interface and internally calls the third-party SDK, translating shapes/units in between.

## 🏗️ Structure
```
PaymentGateway (my interface)
      ↑ implements
RazorpayAdapter
      └── wraps → RazorpayClient (their SDK)
```

## ☕ Java Implementation
```java
interface PaymentGateway { void pay(Booking b, double amountInRupees); }

class RazorpayAdapter implements PaymentGateway {
    private final RazorpayClient client;
    RazorpayAdapter(RazorpayClient client) { this.client = client; }

    public void pay(Booking b, double amountInRupees) {
        client.initiateTxn(b.getId(), (int)(amountInRupees * 100)); // rupees → paise, only here
    }
}
```

## 🏢 SAloon Example
`RazorpayAdapter` and `StripeAdapter` both implement `PaymentGateway` — see [[09 - Real Project LLD/SAloon/01 - Payment]]. `BookingService` never sees Razorpay's SDK types at all.

## 🎯 When should I use it?
Integrating any third-party/legacy API whose interface doesn't match what your code expects and you can't (or shouldn't) change its source.

## ⚠️ When NOT to use it?
Don't adapt an interface you actually own and could just fix directly.

## 🔥 Interview Question
*"Adapter vs Facade — aren't they both 'wrapper' classes?"*
→ Adapter translates one interface into a different, expected interface (fixing incompatibility). Facade simplifies access to a complex subsystem. Different intent, similar wrapper shape.

## 🔗 Related
[[02 - SOLID/04 - DIP]]
[[Facade]]

#pattern/structural
