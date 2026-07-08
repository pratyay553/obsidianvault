---
tags: [dsa, flowchart, decision-tree]
---

# 17. Interview Flowchart — Given a Problem, Now What?

⬅ [[00-Home]] | Prev: [[16-Edge-Case-Checklist]] | Next: [[18-Flashcards]]

```mermaid
graph TD
    A[Read the problem] --> B{Is the array/string sorted, or sortable without breaking constraints?}
    B -->|Yes| C[Binary Search family - 04]
    B -->|No| D{Need a contiguous subarray/substring?}
    D -->|Yes, fixed size K| E[Sliding Window Fixed - 04]
    D -->|Yes, variable size| F[Sliding Window Variable - 04]
    D -->|Yes, sum-related| G[Prefix Sum / Kadane - 03]
    D -->|No| H{Need frequency/counting?}
    H -->|Yes| I[HashMap / Frequency Array - 06]
    H -->|No| J{Need next greater/smaller element?}
    J -->|Yes| K[Monotonic Stack - 07]
    J -->|No| L{Need top-K, k-th, or streaming order?}
    L -->|Yes| M[Heap or Quick Select - 07/05]
    L -->|No| N{Values restricted to range 1..n?}
    N -->|Yes| O[Cyclic Sort - 05]
    N -->|No| P{Dealing with intervals?}
    P -->|Yes| Q[Merge Intervals / Sweep Line - 05]
    P -->|No| R{2D grid / matrix?}
    R -->|Yes, connectivity/regions| S[BFS/DFS Grid or Union Find - 09/08]
    R -->|Yes, rotation/spiral/diagonal| T[Matrix Simulation - 09]
    R -->|No| U{Pattern matching in a string?}
    U -->|Yes| V[KMP / Rabin-Karp / Z-Algorithm - 11]
    U -->|No| W{Palindrome related?}
    W -->|Yes| X[Two Pointer / Expand Around Center / Manacher - 10]
    W -->|No| Y{Prefix/autocomplete over many words?}
    Y -->|Yes| Z[Trie - 11]
    Y -->|No| AA{Repeated range queries with updates?}
    AA -->|Yes| AB[Fenwick Tree / Segment Tree - 08]
    AA -->|No| AC[Likely Greedy or a DP/Graph problem outside array/string scope]
```

## Textual Walkthrough (for when you can't see the diagram)

1. **Sorted?** → Binary Search.
2. **Not sorted, need contiguous range?** → Sliding Window (fixed/variable) or Prefix Sum / Kadane if sum-related.
3. **Need counting/frequency?** → HashMap or fixed-size Frequency Array.
4. **Need "next greater/smaller"?** → Monotonic Stack.
5. **Need top-K / k-th element / streaming order?** → Heap (or Quick Select for one-shot k-th).
6. **Values confined to [1,n] or [0,n-1]?** → Cyclic Sort (O(1) space missing/duplicate tricks).
7. **Intervals?** → Merge Intervals (static) or Sweep Line (overlap counting).
8. **2D grid, connectivity?** → BFS/DFS or Union Find.
9. **2D grid, geometric transform?** → Matrix simulation (rotate/spiral/diagonal/transpose).
10. **String pattern matching?** → KMP (guaranteed), Rabin-Karp (multiple patterns), Z-Algorithm (alternative).
11. **Palindrome?** → Two Pointer (validation) or Expand Around Center / Manacher (substring search).
12. **Prefix-heavy, many words?** → Trie.
13. **Repeated range queries with mutation?** → Fenwick Tree or Segment Tree.
14. **None of the above fit cleanly?** → Reconsider: could be Greedy, DP, or a Graph problem — outside pure array/string pattern scope, but check [[06-Arrays-Hashing-Greedy#Greedy|Greedy]] first since it overlaps often.

Next: [[18-Flashcards]]
