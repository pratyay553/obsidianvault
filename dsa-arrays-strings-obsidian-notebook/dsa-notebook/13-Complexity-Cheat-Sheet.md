---
tags: [dsa, complexity, cheat-sheet]
---

# 13. Complexity Cheat Sheet

⬅ [[00-Home]] | Prev: [[12-Comparison-Tables]] | Next: [[14-Java-Templates]]

| Algorithm | Time | Space | Stable? | In-place? | Interview Frequency | Difficulty |
|---|---|---|---|---|---|---|
| Basic Traversal | O(n) | O(1) | — | ✅ | ⭐⭐⭐⭐⭐ | Easy |
| Prefix Sum | O(n) build / O(1) query | O(n) | — | ❌ | ⭐⭐⭐⭐⭐ | Easy-Medium |
| Difference Array | O(1) update / O(n) build | O(n) | — | ❌ | ⭐⭐⭐ | Medium |
| Kadane | O(n) | O(1) | — | ✅ | ⭐⭐⭐⭐⭐ | Easy-Medium |
| Two Pointer | O(n) | O(1) | — | ✅ | ⭐⭐⭐⭐⭐ | Easy-Medium |
| Sliding Window Fixed | O(n) | O(1)/O(k) | — | ✅ | ⭐⭐⭐⭐ | Medium |
| Sliding Window Variable | O(n) | O(k) | — | ❌ | ⭐⭐⭐⭐⭐ | Medium-Hard |
| Binary Search | O(log n) | O(1) | — | ✅ | ⭐⭐⭐⭐⭐ | Easy-Medium |
| Binary Search on Answer | O(n log range) | O(1) | — | ✅ | ⭐⭐⭐⭐ | Medium-Hard |
| Dutch National Flag | O(n) | O(1) | ❌ | ✅ | ⭐⭐⭐ | Medium |
| Merge Intervals | O(n log n) | O(n) | — | ❌ | ⭐⭐⭐⭐ | Medium |
| Sweep Line | O(n log n) | O(n) | — | ❌ | ⭐⭐⭐ | Medium |
| Cyclic Sort | O(n) | O(1) | ❌ | ✅ | ⭐⭐⭐ | Medium |
| Quick Select | O(n) avg / O(n²) worst | O(1) | ❌ | ✅ | ⭐⭐⭐⭐ | Medium |
| Moore Voting | O(n) | O(1) | — | ✅ | ⭐⭐⭐ | Easy-Medium |
| HashMap ops | O(1) avg | O(n) | — | ❌ | ⭐⭐⭐⭐⭐ | Easy |
| HashSet ops | O(1) avg | O(n) | — | ❌ | ⭐⭐⭐⭐⭐ | Easy |
| Frequency Array | O(n) | O(1) fixed | — | ✅ | ⭐⭐⭐⭐ | Easy |
| Greedy | O(n log n) typical | O(1)/O(n) | — | varies | ⭐⭐⭐⭐ | Medium-Hard |
| Monotonic Stack | O(n) | O(n) | — | ❌ | ⭐⭐⭐⭐ | Medium-Hard |
| Monotonic Queue | O(n) | O(k) | — | ❌ | ⭐⭐⭐ | Hard |
| Deque | O(1) per op | O(n) | — | ❌ | ⭐⭐⭐ | Easy-Medium |
| Heap | O(log n) insert/extract | O(n) | — | ❌ | ⭐⭐⭐⭐⭐ | Medium |
| Bucket Sort | O(n) | O(n) | ✅ | ❌ | ⭐⭐ | Medium |
| Counting Sort | O(n+k) | O(k) | ✅ | ❌ | ⭐⭐ | Easy-Medium |
| Merge Sort | O(n log n) | O(n) | ✅ | ❌ | ⭐⭐⭐ | Medium |
| Quick Sort | O(n log n) avg / O(n²) worst | O(log n) | ❌ | ✅ | ⭐⭐⭐ | Medium |
| Fenwick Tree | O(log n) update/query | O(n) | — | ❌ | ⭐⭐ | Hard |
| Segment Tree | O(log n) update/query | O(4n) | — | ❌ | ⭐⭐ | Hard |
| Sparse Table | O(1) query / O(n log n) build | O(n log n) | — | ❌ | ⭐ | Hard |
| Union Find | O(α(n)) amortized | O(n) | — | ❌ | ⭐⭐⭐ | Medium |
| Matrix Traversal | O(n·m) | O(1) | — | ✅ | ⭐⭐⭐⭐ | Easy-Medium |
| BFS Grid | O(n·m) | O(n·m) | — | ❌ | ⭐⭐⭐⭐ | Medium |
| DFS Grid | O(n·m) | O(n·m) (stack) | — | ❌ | ⭐⭐⭐⭐ | Medium |
| Character Frequency | O(n) | O(1) fixed | — | ✅ | ⭐⭐⭐⭐⭐ | Easy |
| Palindrome (2-pointer) | O(n) | O(1) | — | ✅ | ⭐⭐⭐⭐ | Easy |
| Expand Around Center | O(n²) | O(1) | — | ✅ | ⭐⭐⭐⭐ | Medium |
| Manacher | O(n) | O(n) | — | ❌ | ⭐ | Hard |
| KMP | O(n+m) | O(m) | — | ❌ | ⭐⭐⭐ | Hard |
| Rabin-Karp | O(n+m) avg / O(n·m) worst | O(1) | — | ✅ | ⭐⭐ | Medium-Hard |
| Z-Algorithm | O(n+m) | O(n+m) | — | ❌ | ⭐ | Hard |
| Trie | O(L) insert/search | O(alphabet × nodes) | — | ❌ | ⭐⭐⭐ | Medium |
| Suffix Array | O(n log n) build | O(n) | — | ❌ | ⭐ | Hard |

> [!tip] Legend
> ⭐⭐⭐⭐⭐ = Extremely common (expect at least one per interview loop)
> ⭐ = Rare, mostly senior/competitive-programming-flavored rounds

Next: [[14-Java-Templates]]
