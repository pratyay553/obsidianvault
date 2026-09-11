---
tags: [principle]
---

# YAGNI — You Aren't Gonna Need It

## 🧠 Problem
I built a `NotificationChannelFactory` that supported SMS, Email, Push, **and** Fax "just in case" — SAloon has never sent a fax and never will.

## 🤔 Why does the problem exist
Speculative flexibility has a real cost, and the "future need" I imagined is often wrong anyway — the actual future requirement is usually something I didn't predict.

## 💡 Idea
Don't build for a requirement that doesn't exist yet. Build for what's asked now, and rely on good [[02 - SOLID/01 - OCP|OCP]]-shaped design to extend cheaply *when* the real requirement shows up.

## 🏗️ Structure
```
Build now:            Don't build yet:
SmsNotifier            FaxNotifier
EmailNotifier          WhatsAppNotifier (until actually requested)
```

## ☕ Java Implementation
```java
interface Notifier { void send(String msg); }
class SmsNotifier implements Notifier { ... }
class EmailNotifier implements Notifier { ... }
// Don't pre-build FaxNotifier, TelegramNotifier "to be safe"
```

## 🏢 SAloon Example
[[09 - Real Project LLD/SAloon/02 - Notification]] shipped with SMS + Email only. WhatsApp was added 3 months later by adding one class — because the interface (OCP) was right, not because I pre-built it.

## 🎯 When should I use it?
Every time I catch myself writing "we might need this later."

## ⚠️ When NOT to use it?
Don't use YAGNI to skip designing a clean extension point — YAGNI says don't build the *unused feature*, not "don't leave room to add it cheaply."

## 🔥 Interview Question
*"Isn't this in tension with OCP — design for extension?"*
→ No. OCP says *design so that adding X later is cheap*. YAGNI says *don't build X now if nobody asked for it*. Do both.

## 🔗 Related
[[01 - KISS]]
[[02 - SOLID/01 - OCP]]

#principle
