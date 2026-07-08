---
tags: [dsa, flashcards]
---

# 18b. Flashcards (Continued)

⬅ [[18-Flashcards]] | Next: [[19-Common-Mistakes]]

## Monotonic Stack, Queue, Heap

<details><summary>Q: When to use a Monotonic Stack?</summary>A: "Next greater/smaller element" problems, largest rectangle in histogram.</details>

<details><summary>Q: What do you store in the stack for "Next Greater Element"?</summary>A: Indices (not values) — so you know where to write the answer.</details>

<details><summary>Q: What's the time complexity of a monotonic stack algorithm, and why?</summary>A: O(n) — each element is pushed and popped at most once.</details>

<details><summary>Q: What is a Monotonic Queue used for?</summary>A: Sliding window maximum/minimum in O(n), using a deque of indices.</details>

<details><summary>Q: In a monotonic deque for sliding window max, what direction do values decrease?</summary>A: Front to back — the front always holds the current window's max.</details>

<details><summary>Q: Java's PriorityQueue is a min-heap or max-heap by default?</summary>A: Min-heap. Use `Collections.reverseOrder()` or a custom comparator for max-heap behavior.</details>

<details><summary>Q: How do you find the median of a data stream efficiently?</summary>A: Two heaps — a max-heap for the lower half, a min-heap for the upper half, rebalanced on each insert.</details>

<details><summary>Q: Heap vs Quick Select for "Kth largest" — when to prefer each?</summary>A: Heap for streaming/incremental data; Quick Select for a one-shot O(n) average query on a static array.</details>

## Advanced Data Structures

<details><summary>Q: When to use a Fenwick Tree over plain Prefix Sum?</summary>A: When the array is updated frequently and you need O(log n) update + query instead of O(n) rebuild.</details>

<details><summary>Q: What bit trick powers Fenwick Tree traversal?</summary>A: `i & (-i)` isolates the lowest set bit, used to move between tree indices.</details>

<details><summary>Q: When would you need a Segment Tree instead of a Fenwick Tree?</summary>A: When you need range min/max/gcd (not just sum), or range updates with lazy propagation.</details>

<details><summary>Q: What's the key limitation of a Sparse Table?</summary>A: Only for static (no update) arrays, and only works cleanly for idempotent operations (min/max/gcd), not sum.</details>

<details><summary>Q: Union Find's two key optimizations?</summary>A: Path compression (flatten during find) and union by rank/size (attach smaller tree under larger).</details>

<details><summary>Q: Union Find's amortized time complexity per operation?</summary>A: O(α(n)) — inverse Ackermann function, effectively constant.</details>

## Matrix & Grid

<details><summary>Q: How do you rotate a matrix 90° clockwise in-place?</summary>A: Transpose the matrix, then reverse each row.</details>

<details><summary>Q: How do you group matrix cells lying on the same diagonal?</summary>A: By the value of `i + j` (top-left→bottom-right diagonals) or `i - j` (anti-diagonals).</details>

<details><summary>Q: What's the trick for searching a fully row/column sorted matrix in O(n+m)?</summary>A: Start from the top-right (or bottom-left) corner and move based on comparison with the target.</details>

<details><summary>Q: BFS vs DFS on a grid — which guarantees shortest path?</summary>A: BFS, for unweighted grids.</details>

<details><summary>Q: What's the danger of recursive DFS on a huge grid?</summary>A: Stack overflow — convert to iterative DFS with an explicit stack if the grid is very large.</details>

<details><summary>Q: When to prefer Union Find over BFS/DFS for grid connectivity?</summary>A: When edges/connections are added incrementally over time (dynamic), not a one-shot static scan.</details>

## Strings

<details><summary>Q: What defines an anagram relationship between two strings?</summary>A: Identical character frequency counts.</details>

<details><summary>Q: What's the fastest way to check anagrams for lowercase strings?</summary>A: A 26-length frequency array, incrementing for one string and decrementing for the other, checking for any negative.</details>

<details><summary>Q: What's the time complexity of Expand Around Center?</summary>A: O(n²) — n centers, each expansion up to O(n).</details>

<details><summary>Q: What upgrade gives O(n) longest palindromic substring?</summary>A: Manacher's Algorithm.</details>

<details><summary>Q: What is the "lps" array in KMP?</summary>A: Longest proper prefix that is also a suffix, for each prefix of the pattern — used to skip redundant comparisons on mismatch.</details>

<details><summary>Q: KMP worst-case time complexity?</summary>A: O(n + m), guaranteed — no dependency on hash collisions.</details>

<details><summary>Q: Rabin-Karp's worst-case time complexity, and why?</summary>A: O(n·m) — many hash collisions force full string comparisons at every window.</details>

<details><summary>Q: What is the Z-array (`Z[i]`) in the Z-Algorithm?</summary>A: The length of the longest substring starting at `i` that matches a prefix of the whole string.</details>

<details><summary>Q: What separator trick is used before running Z-algorithm/Manacher on pattern+text?</summary>A: Insert a unique character (e.g. `$` or `#`) not present in either string, to prevent cross-matching.</details>

<details><summary>Q: When is a Trie preferred over a HashMap?</summary>A: When you need prefix-based queries (autocomplete, longest common prefix, word search) efficiently.</details>

<details><summary>Q: What flag does every Trie node need, and why?</summary>A: `isEnd` — to distinguish "this is a complete word" from "this is just a prefix of a longer word."</details>

<details><summary>Q: Why is naive string concatenation in a loop dangerous?</summary>A: `String + String` in a loop is O(n²) total — use `StringBuilder` for O(n).</details>

<details><summary>Q: Why can't you assume `char` == one visible character in Java?</summary>A: Java `char` is a UTF-16 code unit; characters outside the Basic Multilingual Plane use surrogate pairs (2 chars per visible character).</details>

## Complexity & General

<details><summary>Q: Why should you avoid `(lo + hi) / 2` for binary search midpoints?</summary>A: Integer overflow risk for large index values — use `lo + (hi - lo) / 2`.</details>

<details><summary>Q: What's the general rule for choosing HashMap vs sorting-based approaches?</summary>A: HashMap trades O(n) space for O(1) average lookup, turning many O(n²) problems into O(n); sorting trades O(n log n) time for enabling two-pointer/binary-search techniques.</details>

<details><summary>Q: What should you always ask about integer inputs before coding?</summary>A: Could sums/products overflow `int`? Should I use `long`?</details>

<details><summary>Q: What should constraints like "n ≤ 10^5" signal to you?</summary>A: You need O(n log n) or better — O(n²) will likely time out.</details>

<details><summary>Q: What should constraints like "n ≤ 20" signal to you?</summary>A: Exponential/bitmask DP approaches (2^n) may be intended and acceptable.</details>

Next: [[19-Common-Mistakes]]
