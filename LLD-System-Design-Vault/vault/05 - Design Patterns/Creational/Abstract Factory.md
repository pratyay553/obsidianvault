---
tags: [pattern, creational]
---

# Abstract Factory

## 🧠 Problem
SAloon supports multiple "salon themes" for booking receipts/branding — each theme needs a matching `ReceiptFormatter`, `EmailTemplate`, and `SmsTemplate` that are visually consistent. Mixing a "Premium" receipt with a "Basic" email template would look broken.

## 🤔 Why does the problem exist
If each product is created independently, nothing guarantees they're all from the *same* theme/family — I could accidentally mix a Premium receipt with a Basic email footer.

## 💡 Idea
One factory interface produces a whole *matching family* of related objects, guaranteeing they're compatible with each other.

## 🏗️ Structure
```
BrandingFactory (interface)
   ├── createReceiptFormatter()
   ├── createEmailTemplate()
   └── createSmsTemplate()
        ↑ implemented by
   PremiumBrandingFactory / BasicBrandingFactory
```

## ☕ Java Implementation
```java
interface BrandingFactory {
    ReceiptFormatter createReceipt();
    EmailTemplate createEmailTemplate();
}
class PremiumBrandingFactory implements BrandingFactory {
    public ReceiptFormatter createReceipt() { return new PremiumReceiptFormatter(); }
    public EmailTemplate createEmailTemplate() { return new PremiumEmailTemplate(); }
}
class BasicBrandingFactory implements BrandingFactory {
    public ReceiptFormatter createReceipt() { return new BasicReceiptFormatter(); }
    public EmailTemplate createEmailTemplate() { return new BasicEmailTemplate(); }
}
```

## 🏢 SAloon Example
`BrandingFactory` — a salon owner picks a plan tier, and every generated artifact stays visually consistent because they all come from one factory instance. See [[09 - Real Project LLD/SAloon/02 - Notification]].

## 🎯 When should I use it?
When you must guarantee a *set* of created objects are mutually compatible.

## ⚠️ When NOT to use it?
If the "family" constraint doesn't actually exist, this is over-engineering — plain [[Factory]] is enough.

## 🔥 Interview Question
*"Factory Method vs Abstract Factory — what's the actual difference?"*
→ Factory Method creates **one** product via inheritance. Abstract Factory creates a **family** of related products via composition (inject a whole factory object).

## 🔗 Related
[[Factory]]
[[Builder]]

#pattern/creational
