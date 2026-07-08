---
tags: [dsa, tricks]
---

# 15. Common Tricks (100+)

⬅ [[00-Home]] | Prev: [[14-Java-Templates]] | Next: [[16-Edge-Case-Checklist]]

> [!tip] How to use
> Skim category headers first — recognizing "oh, this is a bit-manipulation problem" is half the battle.

## Bit Manipulation Tricks (1–15)

1. `a ^ b ^ b = a` — XOR cancels duplicates → find the single non-duplicate number.
2. `x & (x - 1)` clears the lowest set bit — count set bits, check power of 2.
3. `x & (-x)` isolates the lowest set bit — used in Fenwick Tree indexing.
4. `x & 1` checks odd/even faster than `% 2`.
5. `x << 1` / `x >> 1` for fast ×2 / ÷2.
6. `~x` = bitwise NOT = `-x - 1`.
7. Swap two numbers without a temp var: `a ^= b; b ^= a; a ^= b;` (works, but rarely improves readability — mention only if asked).
8. `Integer.bitCount(x)` — built-in popcount, don't reinvent it.
9. `Integer.numberOfTrailingZeros(x)` / `numberOfLeadingZeros(x)` for fast log2-like ops.
10. Represent a subset with a bitmask for DP over small n (n ≤ ~20).
11. `1 << k` to generate all subsets of size up to `2^n`.
12. Toggle case of an ASCII letter fast: `c ^ 32`.
13. Check if two integers have opposite signs: `(a ^ b) < 0`.
14. `x & (x + 1)` clears trailing 1s (opposite of trick #2).
15. Use XOR to detect the two unique numbers among duplicates: XOR all, split by any set bit.

## ASCII / Character Tricks (16–25)

16. `c - '0'` for digit-to-int; `c - 'a'` for letter-to-index.
17. Frequency array of size 26 (lowercase) or 256 (extended ASCII) beats HashMap for speed.
18. `Character.isDigit`, `isLetter`, `isUpperCase`, `isLowerCase` — avoid manual range checks (more readable, fewer bugs).
19. Build a `boolean[128]` lookup table for "is this character in the target set" — O(1) checks.
20. Convert char array back to String: `new String(charArray)`.
21. `String.valueOf(char)` for single-char to String.
22. Reverse a char array in-place with two pointers instead of `StringBuilder.reverse()` when you must avoid extra allocation.
23. Compare strings case-insensitively: `a.equalsIgnoreCase(b)` (don't manually lowercase both if avoidable).
24. `s.chars()` gives an `IntStream` of char codes — handy for functional-style scans.
25. Use `StringBuilder.setCharAt` instead of rebuilding the whole string for a single-character edit.

## Modulo & Overflow Tricks (26–35)

26. `mid = lo + (hi - lo) / 2` avoids integer overflow (never `(lo + hi) / 2` for large indices).
27. Negative modulo in Java isn't guaranteed non-negative: use `((x % m) + m) % m`.
28. Use `long` accumulators when summing values that could exceed `Integer.MAX_VALUE` (2^31-1 ≈ 2.1 billion).
29. Watch for overflow in `a * b` before comparison — cast to `long` early: `(long) a * b`.
30. `Math.floorDiv` / `Math.floorMod` give mathematically correct (always-consistent-sign) division/modulo.
31. Precompute `MOD = 1_000_000_007` for competitive-style large-number problems.
32. `Integer.MAX_VALUE` = 2147483647; watch for `+1` overflow flipping to negative.
33. When doubling repeatedly (e.g. binary exponentiation), check for overflow before each multiply.
34. Use `Math.addExact`/`Math.multiplyExact` in Java to auto-throw on overflow during debugging.
35. Sum of 1..n formula `n*(n+1)/2` can overflow `int` for large n — use `long`.

## Sentinel & Dummy Value Tricks (36–45)

36. Add a dummy node before the head of a linked list to simplify edge cases (not array-specific, but shows up in "merge" style array problems too).
37. Pad an array with `-∞`/`+∞` sentinels at both ends to avoid boundary checks (e.g., histogram problems).
38. Use `Integer.MIN_VALUE`/`MAX_VALUE` as "no answer yet" sentinels, but be careful when real values could equal them.
39. Append a fake "0-height" bar to the end of a histogram array to force the monotonic stack to flush at the end.
40. Use a `visited` sentinel value baked into the input array itself (e.g., negate values) to save O(n) space — see Negative Marking below.
41. Use a unique separator character (`'$'` or `'#'`) when concatenating two strings for Z-algorithm/Manacher to prevent cross-matching.
42. Initialize `dp[0] = 1` as a base case sentinel in counting DP problems.
43. Use `Optional`-style sentinel objects instead of `null` when `null` is a valid data value itself.
44. For "first negative in every window", use `Integer.MIN_VALUE` to mark "none found yet".
45. Use out-of-range index sentinels (`-1`) to mean "no previous/next element found" in monotonic stack answers.

## In-Place Marking Tricks (46–52)

46. Negative marking: to mark index `i` as "visited" without extra space, set `arr[i] = -arr[i]` (works when all values are known-positive).
47. Cyclic sort marking: place `arr[i] - 1` at index `i` to detect missing/duplicate numbers in O(1) space.
48. Mark visited cells in a grid by mutating the grid value temporarily, then restore afterward if the original grid must be preserved.
49. Use the sign bit of an otherwise-positive array as a "seen" flag (see First Missing Positive, LC 41).
50. In matrix problems, encode two pieces of info in one cell using arithmetic tricks (e.g., `arr[i] += n * newValue` then `arr[i] / n` to decode) when in-place O(1) space is demanded.
51. Swap-based in-place partitioning (Dutch flag, quickselect partition) avoids extra arrays entirely.
52. Use the array itself as a hash set when values are bounded to `[0, n-1]` or `[1, n]`.

## Lazy Deletion / Lazy Evaluation Tricks (53–58)

53. In a heap, instead of removing an arbitrary element (not directly supported), mark it "stale" and skip it when popped ("lazy deletion").
54. Defer sorting until the last possible moment if multiple approaches are being compared during the interview.
55. Use a counter map to represent "still active" elements; decrement instead of removing from the underlying structure.
56. In segment trees, use **lazy propagation** to defer range updates until a node is actually visited.
57. Skip re-computation of unchanged subranges via cached prefix sums instead of recomputing greedily.
58. Union-Find's path compression is a lazy technique — don't flatten paths until a `find()` actually traverses them.

## Traversal Direction Tricks (59–65)

59. Reverse traversal (`for (int i = n - 1; i >= 0; i--)`) is often needed for "distance to the right" or suffix-based computations.
60. Traverse from both ends simultaneously (two pointer) instead of two separate passes.
61. Diagonal traversal: group cells by `i + j` (top-left→bottom-right diagonals) or `i - j` (anti-diagonals).
62. Spiral traversal: shrink 4 boundaries (top/bottom/left/right) each lap.
63. Zig-zag traversal: alternate direction each "row" or "level" (used in level-order tree traversal too).
64. Simulate array rotation via three reversals instead of extra array allocation: reverse whole array, then reverse each part.
65. Traverse a matrix in a "staircase" pattern (start top-right or bottom-left) for sorted-2D-matrix search.

## Coordinate Compression (66–68)

66. When actual values are huge/sparse but only relative order matters, map values to their sorted rank (0, 1, 2, ...) to shrink the working range — enables array/Fenwick-tree tricks on originally huge values.
67. Useful before Fenwick Tree/Segment Tree usage when values aren't naturally small integers.
68. Combine with binary search (`Arrays.binarySearch` on sorted unique values) to get compressed indices in O(log n).

## Two-Pass / Multi-Pass Tricks (69–73)

69. "Product of array except self" — one pass for prefix products, one pass for suffix products, multiply.
70. "Trapping rain water" — precompute `leftMax[]` and `rightMax[]` arrays in two passes (or do it in one pass with two pointers).
71. "Candy distribution" — one left-to-right pass enforcing the right-neighbor rule, one right-to-left pass enforcing the left-neighbor rule, take the max at each index.
72. Validate feasibility on a first pass, then construct the actual answer on a second pass (common in greedy constructions).
73. Count first, then bucket/place — used in bucket sort and counting sort.

## Set/Map Construction Tricks (74–80)

74. Use a `TreeMap`/`TreeSet` when you need sorted iteration order or `floorKey`/`ceilingKey` (nearest ≤/≥ lookups).
75. Use `LinkedHashMap` for insertion-order-preserving maps (e.g., building an LRU cache).
76. `computeIfAbsent(key, k -> new ArrayList<>())` avoids null-checks when building `Map<K, List<V>>`.
77. `merge(key, 1, Integer::sum)` is the cleanest one-liner for frequency counting.
78. Use a `Map<String, List<String>>` with a **sorted-character key** for grouping anagrams.
79. Encode a pair `(a, b)` as a single long key (`(long)a << 32 | b`) to use a `Map<Long, V>` instead of `Map<int[], V>` (arrays don't hash by content!).
80. Never use `int[]`/`List<Integer>` directly as a HashMap key expecting content equality — wrap in a class with `equals`/`hashCode`, use a String key, or use the long-encoding trick above.

## Recursion / Backtracking Tricks (81–86)

81. Always define a clear base case first, then the recursive case — write it top-down.
82. Pass mutable state (like a running list) by reference and **undo the change after recursing** (classic backtracking "add, recurse, remove").
83. Memoize with a `Map<String, Integer>` or 2D `int[][]` array when overlapping subproblems appear (turns exponential into polynomial).
84. Prefer iterative + explicit stack over recursion for very deep inputs (avoid `StackOverflowError`).
85. Prune early: check constraints before recursing, not after, to save wasted calls.
86. Use `Integer.MIN_VALUE`/`MAX_VALUE` as initial "worst" values when tracking best result across recursive calls.

## Miscellaneous High-Value Tricks (87–100+)

87. `Arrays.asList(arr)` on a primitive array creates a `List<int[]>` of ONE element, not what you want — box first or use streams.
88. `Collections.unmodifiableList` / `List.of` to signal (and enforce) immutability of results.
89. `Arrays.fill(arr, val)` to quickly initialize a whole array — cleaner than manual loops.
90. `Arrays.equals(a, b)` for array content comparison (`==` compares references!).
91. `System.arraycopy` for fast bulk array copying — faster than manual loops for large arrays.
92. `Collections.reverseOrder()` for one-line max-heaps / descending comparators.
93. `Comparator.comparing(...).thenComparing(...).reversed()` chains for multi-key sorting.
94. Use `Integer.compare(a, b)` instead of `a - b` in comparators — avoids overflow for extreme values.
95. `Math.max`/`Math.min` chains read better than nested ternaries for 3+ way comparisons.
96. Precompute powers of 2 or factorials in a lookup array if you need them repeatedly.
97. `String.join(", ", list)` for clean output formatting instead of manual loop + StringBuilder.
98. Binary search inside a "check feasibility" function is a common **meta-pattern** — recognize when a brute-force O(n²) can become O(n log n) this way.
99. When stuck, ask: "can I sort first?" — many O(n²) problems become O(n log n) once sorted.
100. When stuck, ask: "can I use extra space (HashMap/Set) to trade memory for time?" — turns O(n²) into O(n).
101. When stuck, ask: "is there a monotonic property I can exploit with two pointers or a stack?"
102. Always restate the problem's constraints out loud — they often hint at the expected complexity (n ≤ 20 → exponential OK; n ≤ 10^5 → need O(n log n) or better).
103. If asked for O(1) space with a "corrupt the input" allowed, in-place marking tricks (46–52) are your friend.
104. Draw the example by hand before coding — most sliding-window/two-pointer bugs are caught this way.

Next: [[16-Edge-Case-Checklist]]
