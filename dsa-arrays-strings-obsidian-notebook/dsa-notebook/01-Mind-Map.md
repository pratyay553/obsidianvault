---
tags: [dsa, mindmap]
---

# 1. Mind Map

⬅ [[00-Home]] | Next: [[02-Pattern-Recognition]]

## Arrays

```mermaid
graph LR
    ARR[Arrays] --> T[Traversal]
    ARR --> PS[Prefix Sum]
    ARR --> DA[Difference Array]
    ARR --> TP[Two Pointers]
    ARR --> SW[Sliding Window]
    ARR --> BS[Binary Search]
    ARR --> SRT[Sorting]
    ARR --> GR[Greedy]
    ARR --> HSH[Hashing]
    ARR --> MS[Monotonic Stack/Queue]
    ARR --> HP[Heap]
    ARR --> KAD[Kadane]
    ARR --> CS[Cyclic Sort]
    ARR --> MAT[Matrix]
    ARR --> ADV[Advanced: Fenwick/Segment/Sparse/Union-Find]
```

## Strings

```mermaid
graph LR
    STR[Strings] --> STT[Traversal]
    STR --> FC[Frequency Count]
    STR --> SWS[Sliding Window]
    STR --> KMP[KMP]
    STR --> RK[Rabin Karp]
    STR --> Z[Z Algorithm]
    STR --> TR[Trie]
    STR --> RH[Rolling Hash]
    STR --> MAN[Manacher]
    STR --> PAL[Palindrome]
    STR --> SM[String Matching]
```

> [!tip] Mnemonic — "PALS THINK BIG" for array patterns
> **P**refix sum, **A**rray traversal, **L**inear scan, **S**liding window,
> **T**wo pointer, **H**ashing, **I**ntervals, **N**ested search (binary),
> **K**adane, **B**ucket/counting sort, **I**ncreasing/decreasing stack, **G**reedy
Duplicate       → Frequency
Missing         → Frequency
Anagram         → Character Frequency
Most frequent   → Frequency
Unique element  → Frequency
Count elements  → Frequency
Repeated values → Frequency
## One Diagram to Rule Them All — Pattern Selector

```mermaid
graph TD
    Start[Read the problem] --> Q1{Is input sorted or can be sorted?}
    Q1 -->|Yes| BS[Binary Search family]
    Q1 -->|No| Q2{Contiguous subarray/substring?}
    Q2 -->|Yes, fixed size| SWF[Sliding Window Fixed]
    Q2 -->|Yes, variable size| SWV[Sliding Window Variable]
    Q2 -->|Sum related| PS[Prefix Sum / Kadane]
    Q2 -->|No| Q3{Need frequency/count?}
    Q3 -->|Yes| HM[HashMap/Frequency Array]
    Q3 -->|No| Q4{Need next greater/smaller?}
    Q4 -->|Yes| MST[Monotonic Stack]
    Q4 -->|No| Q5{Need top-K / streaming order?}
    Q5 -->|Yes| HEAP[Heap]
    Q5 -->|No| Q6{In-place range 1..n?}
    Q6 -->|Yes| CYC[Cyclic Sort]
    Q6 -->|No| Q7{Intervals?}
    Q7 -->|Yes| MI[Merge Intervals / Sweep Line]
    Q7 -->|No| GEN[Greedy / DP / Graph — outside this vault's scope]
```

Next: [[02-Pattern-Recognition]]
