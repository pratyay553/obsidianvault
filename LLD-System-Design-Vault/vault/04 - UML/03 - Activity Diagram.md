---
tags: [uml]
---

# Activity Diagram

## 🧠 Problem
Booking a slot at SAloon involves several conditional steps — check availability, apply discount if eligible, charge payment, send notification — and I keep losing track of the branches in prose.

## 🤔 Why does the problem exist
A business process with decision points is basically a flowchart. Describing it linearly in text hides the branches.

## 💡 Idea
Model the *process*, not the objects — start node, activities (boxes), decision diamonds, end node.

## 🏗️ Structure
```
(start) → Check availability → {available?}
              no → Show "slot taken" → (end)
              yes → Apply discount? → Charge payment → {success?}
                        no → Show error → (end)
                        yes → Confirm booking → Notify customer → (end)
```

## 🏢 SAloon Example
This is the actual checkout flow diagram used while designing [[09 - Real Project LLD/SAloon/00 - Booking]] before writing any class — it's what exposed that "apply discount" needed to happen *before* charging.

## 🎯 When should I use it?
Multi-step business processes with real branching logic, before you've decided on classes yet.

## ⚠️ When NOT to use it?
Once the process is simple/linear, just write the steps as a numbered list.

## 🔥 Interview Question
*"How do you decide the order of operations in a multi-step interview problem?"*
→ Sketch it as an activity diagram first — it forces you to spot ordering dependencies before committing to a class design.

## 🔗 Related
[[01 - Sequence Diagram]]
[[02 - State Diagram]]

#uml
