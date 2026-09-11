---
tags: [pattern, structural]
---

# Decorator

## 🧠 Problem
A SAloon booking's price needs: base price, optionally +tax, optionally +peak-hour surcharge, optionally -loyalty discount — in any combination. Subclassing every combination explodes.

## 🤔 Why does the problem exist
Combining N independent optional behaviors via subclassing needs up to 2^N subclasses.

## 💡 Idea
Wrap the base object in layers, each adding one behavior and delegating to the object it wraps. Stack however many you need, in any order.

## 🏗️ Structure
```
PriceComponent (interface)
   BasePrice
      ↑ wrapped by
   TaxDecorator
      ↑ wrapped by
   SurchargeDecorator
```

## ☕ Java Implementation
```java
interface PriceComponent { double getPrice(); }

class BasePrice implements PriceComponent {
    private double amount;
    public double getPrice() { return amount; }
}

abstract class PriceDecorator implements PriceComponent {
    protected PriceComponent wrapped;
    PriceDecorator(PriceComponent w) { wrapped = w; }
}
class TaxDecorator extends PriceDecorator {
    TaxDecorator(PriceComponent w) { super(w); }
    public double getPrice() { return wrapped.getPrice() * 1.18; }
}
class PeakHourSurcharge extends PriceDecorator {
    PeakHourSurcharge(PriceComponent w) { super(w); }
    public double getPrice() { return wrapped.getPrice() + 100; }
}

PriceComponent price = new TaxDecorator(new PeakHourSurcharge(new BasePrice()));
```

## 🏢 SAloon Example
Exactly this stack — see [[09 - Real Project LLD/SAloon/04 - Pricing]]. Adding a new surcharge type later is one new decorator class, zero changes to existing ones.

## 🎯 When should I use it?
Optional, combinable responsibilities on individual objects, where subclassing would multiply combinatorially.

## ⚠️ When NOT to use it?
If there are only 2 fixed combinations that will never grow, a couple of subclasses are simpler.

## 🔥 Interview Question
*"How is Decorator different from just adding fields/flags to the base class?"*
→ Flags mean the base class must know about every possible add-on. Decorator means the base class knows nothing about tax or surcharges — each concern lives in its own class, added externally.

## 🔗 Related
[[02 - SOLID/01 - OCP]]
[[Proxy]]

#pattern/structural
