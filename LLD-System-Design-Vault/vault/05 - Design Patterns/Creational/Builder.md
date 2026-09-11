---
tags: [pattern, creational]
---

# Builder

## 🧠 Problem
`new Booking(customerId, stylistId, serviceId, slotTime, null, false, null, discountCode, null)` — half these fields are optional, I can't tell what `false` and `null` mean without checking the constructor signature every time.

## 🤔 Why does the problem exist
Telescoping constructors (or one giant constructor with many optional params) are unreadable and error-prone — easy to swap two `String` args by accident and the compiler won't catch it.

## 💡 Idea
Build the object step by step through named methods, then call `.build()` once everything required is set — and make the final object immutable.

## 🏗️ Structure
```
Booking.Builder()
   .customer(c)
   .slot(s)
   .discountCode(code)   // optional — just skip if unused
   .build()
```

## ☕ Java Implementation
```java
public class Booking {
    private final Customer customer;
    private final Slot slot;
    private final String discountCode; // optional

    private Booking(Builder b) {
        this.customer = b.customer; this.slot = b.slot; this.discountCode = b.discountCode;
    }

    public static class Builder {
        private Customer customer;
        private Slot slot;
        private String discountCode;
        public Builder customer(Customer c) { this.customer = c; return this; }
        public Builder slot(Slot s) { this.slot = s; return this; }
        public Builder discountCode(String code) { this.discountCode = code; return this; }
        public Booking build() {
            if (customer == null || slot == null) throw new IllegalStateException("missing required fields");
            return new Booking(this);
        }
    }
}
```

## 🏢 SAloon Example
`Booking.Builder` — see [[09 - Real Project LLD/SAloon/00 - Booking]]. `.build()` also validates required fields, so an invalid `Booking` can never exist.

## 🎯 When should I use it?
Objects with several optional fields, or where you want immutability while validating across multiple fields before construction succeeds.

## ⚠️ When NOT to use it?
A class with 2-3 required fields and no optional ones — just use a normal constructor.

## 🔥 Interview Question
*"Builder vs Factory — when do you pick which?"*
→ Factory answers "*which class* do I instantiate?" Builder answers "*how do I assemble* this one complex object?" A factory can internally use a builder.

## 🔗 Related
[[Factory]]
[[Abstract Factory]]

#pattern/creational
