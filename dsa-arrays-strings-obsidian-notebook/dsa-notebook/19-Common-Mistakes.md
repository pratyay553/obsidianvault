---
tags: [dsa, mistakes]
---

# 19. Common Interview Mistakes (Per Algorithm)

⬅ [[00-Home]] | Prev: [[18b-Flashcards-Continued]] | Next: [[20-LeetCode-Roadmap]]

| Algorithm | Common Mistakes |
|---|---|
| Binary Search | Infinite loop (wrong `lo`/`hi` update); mid overflow; wrong boundary condition (`<` vs `<=`); not distinguishing lower/upper bound |
| Kadane | Initializing best sum to 0 (fails on all-negative); resetting running sum to 0 instead of comparing |
| Prefix Sum | Forgetting to seed HashMap with `{0:1}`; off-by-one between `prefix[i]` and `prefix[i+1]` |
| Two Pointer | Using on unsorted array; wrong loop condition (`<` vs `<=`); not skipping duplicates in 3Sum-style problems |
| Sliding Window | Shrinking with `if` instead of `while`; forgetting to remove zero-count keys from the frequency map; updating `best` at the wrong point |
| Dutch National Flag | Incrementing `mid` after a high-swap (unchecked value skipped) |
| Cyclic Sort | Infinite loop from missing equality guard on duplicate values |
| Quick Select / Quick Sort | Deterministic pivot on sorted input → O(n²); off-by-one converting "kth largest" to 0-indexed target |
| Moore Voting | Skipping the verification pass when majority isn't guaranteed |
| Merge Intervals | Forgetting to sort first; wrong overlap boundary (`<` vs `<=`) |
| Monotonic Stack | Storing values instead of indices; wrong stack order (increasing vs decreasing) |
| Monotonic Queue | Wrong window-expiry boundary check; forgetting result only starts once window is full |
| Heap | Forgetting Java's PriorityQueue is min-heap by default; assuming iteration order is sorted |
| Fenwick/Segment Tree | 0-indexed vs 1-indexed confusion; array too small (`4n` for segment tree) |
| Union Find | Forgetting path compression or union by rank — degrades to O(n) per operation |
| Matrix Rotation | Starting the transpose inner loop at `j=0` instead of `j=i+1` (double-swap bug) |
| Spiral Matrix | Missing extra boundary guards on last row/column (double-counts on 1×n or n×1 matrices) |
| BFS Grid | Marking visited on `poll` instead of `offer` (duplicate enqueues, exponential blowup) |
| DFS Grid | Stack overflow on huge grids (recursion too deep); forgetting to mark visited before recursing |
| KMP | Off-by-one in lps table; forgetting to fall back `j = lps[j-1]` on mismatch |
| Rabin-Karp | Negative modulo not corrected; skipping full-string verification after hash match |
| Z-Algorithm | Forgetting the unique separator between pattern and text |
| Trie | Forgetting `isEnd` flag; array size mismatch for non-lowercase input |
| Manacher | Forgetting to handle both odd and even length palindromes uniformly (via transformed string) |
| Greedy (general) | Assuming greedy works without proving it; sorting by the wrong key |
| HashMap/HashSet | Checking `containsKey` after inserting instead of before (self-matching bug); using array/list content as key without proper equals/hashCode |
| StringBuilder | Not using it at all — naive `String +=` in a loop is O(n²) |
| Recursion/Backtracking | Forgetting to undo state after recursing; missing/incorrect base case |

Next: [[20-LeetCode-Roadmap]]
