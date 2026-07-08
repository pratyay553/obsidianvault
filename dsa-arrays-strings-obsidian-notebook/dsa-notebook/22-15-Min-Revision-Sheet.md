---
tags: [dsa, revision, quick]
---

# 22. 15-Minute Revision Sheet

⬅ [[00-Home]] | Prev: [[21-One-Page-Revision-Sheet]] | Next: [[23-Final-Interview-Checklist]]

> [!warning] Read this and nothing else if time is almost up.

## The 8 Questions You Must Be Able to Answer Instantly

1. **Contiguous + sum →** Prefix Sum or Kadane.
2. **Substring/subarray with a shrinking/growing condition →** Sliding Window.
3. **Sorted array, pair/triplet →** Two Pointer.
4. **Kth largest/smallest, top-K →** Heap (streaming) or Quick Select (one-shot).
5. **Next greater/smaller element →** Monotonic Stack.
6. **Values in range [1,n] →** Cyclic Sort.
7. **Overlapping intervals →** Merge Intervals (static) or Sweep Line (counting).
8. **Grid connectivity →** BFS/DFS or Union Find.

## 5 Bugs That Cost People Interviews

1. `mid = (lo + hi) / 2` → overflow. Use `lo + (hi - lo) / 2`.
2. Kadane initialized to `0` → wrong on all-negative arrays.
3. Sliding window shrink using `if` instead of `while`.
4. `PriorityQueue` in Java is a **min-heap** by default.
5. Naive `String +=` in a loop is O(n²) — use `StringBuilder`.

## 5 Templates to Have Cold

```java
// Binary search midpoint
int mid = lo + (hi - lo) / 2;

// Kadane
maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
maxSoFar = Math.max(maxSoFar, maxEndingHere);

// Sliding window shrink
while (/* invalid */) { /* remove left */ left++; }

// Monotonic stack (next greater)
while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) result[stack.pop()] = arr[i];
stack.push(i);

// Frequency map one-liner
freq.merge(c, 1, Integer::sum);
```

## Last-Minute Reminders

- State your **assumptions** out loud (sorted? duplicates allowed? negative numbers?).
- Mention **time/space complexity** before and after coding.
- Dry run on a **small example** before declaring done.
- If stuck, say the pattern-recognition clue out loud — it often unlocks the approach.
- It's fine to start with brute force, then optimize — narrate the upgrade path.

Next: [[23-Final-Interview-Checklist]]
