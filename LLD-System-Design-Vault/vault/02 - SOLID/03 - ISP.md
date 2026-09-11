---
tags: [solid]
---

# Interface Segregation Principle

## 🧠 Problem
`StaffMember` interface has `cutHair()`, `colorHair()`, `manageFrontDesk()`. A new `Receptionist` class is forced to implement `cutHair()` — which makes no sense for it.

## 🤔 Why does the problem exist
A single fat interface forces every implementer to carry methods it doesn't need, which means empty/throwing stub implementations, and any change to the interface forces *every* implementer to change even if irrelevant to them.

## 💡 Idea
Many small, role-specific interfaces beat one fat interface. Implement only what applies to you.

## 🏗️ Structure
```
Bad:                    Good:
StaffMember              HairStylist       FrontDeskStaff
 ├─ cutHair()             └─ cutHair()      └─ manageFrontDesk()
 ├─ colorHair()                              
 └─ manageFrontDesk()
```

## ☕ Java Implementation
```java
interface HairStylist { void cutHair(); void colorHair(); }
interface FrontDeskStaff { void manageFrontDesk(); }

class Receptionist implements FrontDeskStaff { public void manageFrontDesk() {} }
class Stylist implements HairStylist { public void cutHair() {} public void colorHair() {} }
```

## 🏢 SAloon Example
[[09 - Real Project LLD/SAloon/00 - Booking]] assigns a slot to whoever implements `ServiceProvider` for that specific service — a `Stylist` and a `Masseuse` don't share an interface method neither of them needs.

## 🎯 When should I use it?
Whenever I catch myself writing an empty method body or a `throw new UnsupportedOperationException()` just to satisfy an interface — that's the signal to split it.

## ⚠️ When NOT to use it?
Splitting a 2-method interface into two 1-method interfaces "just in case" adds indirection with no payoff — only split along real, observed role boundaries.

## 🔥 Interview Question
*"How is ISP different from SRP?"*
→ SRP is about a **class** having one reason to change. ISP is about an **interface** not forcing implementers to depend on methods they don't use.

## 🔗 Related
[[00 - SRP]]
[[02 - LSP]]

#solid
