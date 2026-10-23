---
tags: [utilities, datetime, interview]
---

# Utilities & Date/Time Handling

Related: [[01 - Spring Scheduling Basics]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
Every scheduler eventually needs to compare "today" against stored dates (birthdays, due dates, report windows) — and date/time handling is a classic source of subtle bugs (timezones!). This module also keeps a general `Utils` grab-bag for shared helper methods.

## 🔑 Modern Date/Time API (`java.time`, since Java 8)

| Class | Use for |
|---|---|
| `LocalDateTime` | Date + time, **no timezone** — good for "wall clock" values like "8:00 AM every day" |
| `ZonedDateTime` | Date + time + timezone — needed when comparing/scheduling across regions |
| `DateTimeFormatter` | Parsing/formatting dates to/from strings (thread-safe, unlike old `SimpleDateFormat`) |

```java
LocalDateTime now = LocalDateTime.now();
ZonedDateTime istNow = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formatted = now.format(fmt);
LocalDateTime parsed = LocalDateTime.parse("2026-09-24 10:30:00", fmt);
```

### ⚠️ Why `SimpleDateFormat` is still in the reference sheet but risky
`SimpleDateFormat` is **not thread-safe** — sharing one static instance across multiple scheduled jobs running concurrently (especially relevant now with [[02 - Virtual Threads and SimpleAsyncTaskScheduler]] enabling real concurrency) can silently corrupt parsed dates. `DateTimeFormatter` (java.time) is immutable and thread-safe, so it's the modern, safer choice — legacy code often still has `SimpleDateFormat` simply because it predates Java 8.

## 🛠️ A Realistic "Birthday Today" Check

```java
public boolean isBirthdayToday(LocalDate dob, ZoneId zone) {
    LocalDate today = LocalDate.now(zone);
    return dob.getMonthValue() == today.getMonthValue()
        && dob.getDayOfMonth() == today.getDayOfMonth();
}
```
Passing an explicit `ZoneId` (rather than relying on the server's default timezone) matters a lot in a multi-region deployment — "today" in UTC and "today" in `Asia/Kolkata` can differ by several hours around midnight.

## ❓ Interview Q&A

**Q: Why is `SimpleDateFormat` dangerous in a multi-threaded scheduler?**
A: It's stateful and mutable internally, so if a single shared instance is reused across concurrently-running scheduled jobs (or virtual threads), you can get corrupted/incorrect parsed results due to race conditions. `DateTimeFormatter` is immutable and thread-safe, avoiding this entirely.

**Q: `LocalDateTime` vs `ZonedDateTime` — when do you need the latter?**
A: Use `LocalDateTime` when the timezone is implicit/irrelevant (e.g., "run at 8am server time"). Use `ZonedDateTime` whenever you're comparing times across regions, storing timestamps that need to be unambiguous globally, or scheduling something meant to fire at a specific *local* time in a specific region regardless of server location.

**Q: How would you avoid a scheduled job firing at the wrong time after a Daylight Saving Time transition?**
A: Use `ZonedDateTime`/`ZoneId`-aware scheduling rather than raw offsets, and prefer cron expressions evaluated against a specific `ZoneId` (Spring's `@Scheduled(cron = "...", zone = "America/New_York")`) so the trigger tracks local wall-clock time correctly through DST shifts.
