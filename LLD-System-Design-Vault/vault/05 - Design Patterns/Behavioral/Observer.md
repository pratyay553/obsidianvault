---
tags: [pattern, behavioral]
---

# Observer

## 🧠 Problem
When a SAloon booking gets confirmed, three unrelated things need to happen: notify the customer, update the stylist's calendar, and log an analytics event. I don't want `BookingService.confirm()` to directly call all three every time a fourth thing gets added later.

## 🤔 Why does the problem exist
If `BookingService` calls each dependent action directly, it needs to know about *every* interested party, and adding a new one means editing `BookingService` again — an OCP violation.

## 💡 Idea
`Booking` (the subject) keeps a list of `Observer`s and notifies them all when its state changes. It doesn't know or care what each observer does.

## 🏗️ Structure
```
Booking (subject)
   - observers: List<BookingObserver>
   + confirm() { ...; notifyAll(); }

BookingObserver (interface)
   ├── NotificationObserver
   ├── CalendarSyncObserver
   └── AnalyticsObserver
```

## ☕ Java Implementation
```java
interface BookingObserver { void onConfirmed(Booking b); }

class Booking {
    private List<BookingObserver> observers = new ArrayList<>();
    void subscribe(BookingObserver o) { observers.add(o); }
    void confirm() {
        this.status = CONFIRMED;
        observers.forEach(o -> o.onConfirmed(this));
    }
}

class NotificationObserver implements BookingObserver {
    public void onConfirmed(Booking b) { /* send SMS/email */ }
}
```

## 🏢 SAloon Example
See [[09 - Real Project LLD/SAloon/02 - Notification]] — adding "sync to Google Calendar" later meant adding one new observer class, zero edits to `Booking.confirm()`.

## 🎯 When should I use it?
Event-driven fan-out where the number of interested parties can grow, and the subject shouldn't need to know their concrete types.

## ⚠️ When NOT to use it?
If there's exactly one thing that always happens on this event and it'll never grow, a direct method call is simpler and easier to trace/debug than an indirect notify chain.

## 🔥 Interview Question
*"What can go wrong with Observer in practice?"*
→ Silent failures (one observer throwing can block the rest unless you isolate exceptions per observer) and hard-to-trace call chains for debugging — worth mentioning both as trade-offs, not just benefits.

## 🔗 Related
[[../../01 - OOP Fundamentals/03 - Polymorphism]]
[[Command]]

#pattern/behavioral
