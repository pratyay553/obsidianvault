---
tags: [saloon, real-project]
---

# SAloon — Notification Module

## 🧠 Problem
Notify a customer of booking confirmation/cancellation over whichever channel they prefer (SMS, Email, and later WhatsApp) — without `BookingService` growing an `if/else` per channel, and without every downstream reaction to a booking event (notification, calendar sync, analytics) being hardcoded into `Booking` itself.

## 🤔 Why does the problem exist
The first version had `Booking.confirm()` directly calling `smsService.send()` — adding calendar sync later meant editing `Booking` again, an [[../../02 - SOLID/01 - OCP|OCP]] violation.

## 💡 Idea
`Booking` raises a [[../../06 - LLD Building Blocks/05 - Domain Events|Domain Event]] on confirmation. Multiple `BookingObserver`s ([[../../05 - Design Patterns/Behavioral/Observer|Observer]]) subscribe and react independently. Which concrete `Notifier` to use is decided by a [[../../05 - Design Patterns/Creational/Factory|Factory]] based on customer preference. A [[../../05 - Design Patterns/Creational/Abstract Factory|BrandingFactory]] keeps the receipt/email/SMS templates visually consistent per salon plan tier.

## 🏗️ Structure
```
Booking.confirm() → notifies → List<BookingObserver>
   ├── NotificationObserver → NotifierFactory.create(customer.channel) → SmsNotifier/EmailNotifier
   ├── CalendarSyncObserver
   └── AnalyticsObserver

BrandingFactory (per plan tier) → ReceiptFormatter + EmailTemplate + SmsTemplate (all matching)
```

## ☕ Java Implementation
```java
interface Notifier { void send(String msg); }
class NotifierFactory {
    static Notifier create(Channel channel) {
        return switch (channel) {
            case SMS -> new SmsNotifier();
            case EMAIL -> new EmailNotifier();
            case WHATSAPP -> new WhatsAppNotifier(); // added later, zero changes elsewhere
        };
    }
}

class NotificationObserver implements BookingObserver {
    public void onConfirmed(Booking b) {
        Notifier n = NotifierFactory.create(b.getCustomer().getPreferredChannel());
        n.send("Your booking is confirmed for " + b.getSlot());
    }
}
```

## Why this survived a real requirement change
WhatsApp support was added 3 months after launch — one new `WhatsAppNotifier` class, one new `switch` case. Zero changes to `Booking`, `BookingFacade`, or any observer. That's [[../../02 - SOLID/01 - OCP|OCP]] actually paying off, not just a theoretical benefit.

## Every pattern this module uses
- [[../../05 - Design Patterns/Behavioral/Observer]] — decoupled reactions to confirmation
- [[../../05 - Design Patterns/Creational/Factory]] — channel selection
- [[../../05 - Design Patterns/Creational/Abstract Factory]] — consistent branding family
- [[../../06 - LLD Building Blocks/05 - Domain Events]] — `BookingCancelledEvent`, `BookingConfirmedEvent`

## 🔥 Interview Question
*"What happens if one observer throws an exception — should it block the others?"*
→ No — each observer's `onConfirmed()` call should be wrapped so one failure (say, SMS provider down) doesn't stop calendar sync or analytics from running. Log the failure, keep going.

## 🔗 Related
[[00 - Booking]]
[[../../05 - Design Patterns/Behavioral/Observer]]

#saloon
