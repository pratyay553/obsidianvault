---
tags: [dsa, edge-cases, checklist]
---

# 16. Edge Case Checklist

⬅ [[00-Home]] | Prev: [[15-Common-Tricks]] | Next: [[17-Interview-Flowchart]]

> [!warning] Before you say "I'm done" — run through this list out loud.

## Universal Array/String Checklist

- [ ] **Empty input?** (`[]`, `""`) — does your code crash or return a sensible default?
- [ ] **Single element?** — off-by-one bugs love this case.
- [ ] **Two elements?** — smallest case where pointers/windows actually move.
- [ ] **All duplicates?** — breaks naive `distinct`/`equals`-based logic.
- [ ] **Negative numbers?** — breaks sentinel tricks (`0` or `-1` as "empty" marker), breaks Kadane if initialized wrong.
- [ ] **Zero(s) in the array?** — breaks product-based tricks (division to "remove" an element fails).
- [ ] **Integer overflow?** — sums, products, `mid` calculation; use `long` where needed.
- [ ] **Integer.MIN_VALUE / MAX_VALUE as actual input?** — sentinels can collide with real data.
- [ ] **Large input size (10^5–10^7)?** — confirms O(n log n) or better is required; O(n²) will TLE.
- [ ] **Already sorted / reverse sorted?** — worst case for naive QuickSort; best/worst case for greedy.
- [ ] **Unsorted input when a pattern assumes sorted?** — did you sort first, or pick the wrong pattern?
- [ ] **Case sensitivity (strings)?** — "Anagram" vs "anagram" — confirm requirements.
- [ ] **Repeated characters (strings)?** — breaks naive frequency logic if not handled with counts.
- [ ] **Unicode/emoji/surrogate pairs?** — `char` ≠ code point; ask if this matters.
- [ ] **Circular array?** — index wrap-around via `% n`; double-length trick for monotonic stack.
- [ ] **Window size 1?** — sliding window degenerates to basic traversal — still correct?
- [ ] **Window size = n (whole array)?** — should reduce to the "global" answer.
- [ ] **k = 0?** — many "top-k"/"first-k" problems need this guarded.
- [ ] **k > n?** — should you cap k, or is it guaranteed valid by constraints?
- [ ] **All elements identical?** — breaks binary search assumptions (no unique target), breaks distinct-based logic.
- [ ] **No valid answer exists?** — what do you return: `-1`, `null`, empty list, throw?
- [ ] **Multiple valid answers?** — confirm if any valid answer is acceptable or a specific one is required (e.g., leftmost).
- [ ] **Very long single word / no whitespace at all (strings)?** — breaks naive `split`-based logic.
- [ ] **Leading/trailing whitespace?** — trim before processing if relevant.
- [ ] **Mutation of input allowed?** — clarify if you can sort/modify in-place or must preserve original.

## Pattern-Specific Reminders

| Pattern         | Extra Edge Cases                                                                |
| --------------- | ------------------------------------------------------------------------------- |
| Binary Search   | target not in array; array of size 0/1; duplicates needing lower/upper bound    |
| Sliding Window  | condition never satisfiable (return 0/-1); window larger than array             |
| Kadane          | all negative numbers (must not default to 0)                                    |
| Two Pointer     | no valid pair; all same value                                                   |
| Monotonic Stack | strictly increasing/decreasing input (empty result / full flush at end)         |
| Heap            | k > array size; all equal priorities                                            |
| Merge Intervals | single interval; fully nested intervals; touching but not overlapping intervals |
| Cyclic Sort     | duplicate values causing infinite swap loop without an equality guard           |
| Matrix          | 1×1 matrix; single row; single column; non-square matrix                        |
| Trie            | empty string insert/search; prefix equals a full word already in the trie       |
| KMP/Z-Algorithm | pattern longer than text; pattern with all identical characters                 |
| Union Find      | self-union; union of already-connected nodes                                    |

Next: [[17-Interview-Flowchart]]
