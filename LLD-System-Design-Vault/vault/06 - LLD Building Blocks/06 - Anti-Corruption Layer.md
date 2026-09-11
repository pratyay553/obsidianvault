---
tags: [building-block]
---

# Anti-Corruption Layer

## 🧠 Problem
SAloon's core domain model calls a stylist's calendar-blocking concept a `Slot`. The third-party calendar sync API calls the same concept an `Event` with a totally different field layout (start/end as separate ISO strings, timezone offsets, recurrence rules). Letting `Event`'s shape leak into `Booking` logic would corrupt the domain model.

## 🤔 Why does the problem exist
Directly binding domain logic to an external system's data shapes means every quirk of that external API (naming, units, edge cases) leaks into your own model — and multiplies if you integrate a second, differently-shaped external system.

## 💡 Idea
Put a translation layer at the boundary that converts the external system's model into your own clean domain model (and back), so nothing outside that layer ever sees the external shape.

## 🏗️ Structure
```
External CalendarAPI (Event, ISO strings, tz offsets)
        ↓
CalendarAntiCorruptionLayer  ← all translation logic lives here
        ↓
Domain model (Slot, clean value objects)
```

## ☕ Java Implementation
```java
class CalendarAntiCorruptionLayer {
    Slot toDomain(ExternalCalendarEvent event) {
        Instant start = Instant.parse(event.startIso()).plus(event.tzOffsetMinutes(), MINUTES);
        return new Slot(start, start.plus(event.durationMinutes(), MINUTES));
    }
}
```

## 🏢 SAloon Example
The calendar sync integration (Google Calendar) goes through exactly this layer — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. `Booking`/`Slot` logic never sees an `ExternalCalendarEvent`.

## 🎯 When should I use it?
Integrating any external system whose model doesn't cleanly map to your own domain concepts — especially legacy systems or third-party APIs you don't control.

## ⚠️ When NOT to use it?
If the external system's model *is* basically your domain model already (same shape, same semantics), a full ACL is unnecessary ceremony — a thin [[../05 - Design Patterns/Structural/Adapter|Adapter]] is enough.

## 🔥 Interview Question
*"How is this different from just using the Adapter pattern?"*
→ Adapter usually translates a single interface's method signatures. An Anti-Corruption Layer is a broader architectural boundary — it can involve multiple adapters, translation of whole data models, and exists specifically to protect the *domain model's integrity*, not just interface compatibility.

## 🔗 Related
[[../05 - Design Patterns/Structural/Adapter]]
[[../02 - SOLID/04 - DIP]]

#building-block
