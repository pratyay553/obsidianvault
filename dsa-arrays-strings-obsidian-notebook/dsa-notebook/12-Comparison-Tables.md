---
tags: [dsa, comparison-tables]
---

# 12. Comparison Tables

⬅ [[00-Home]] | Prev: [[11-Strings-Advanced]] | Next: [[13-Complexity-Cheat-Sheet]]

## Sliding Window vs Two Pointer

| | Sliding Window | Two Pointer |
|---|---|---|
| Structure | Contiguous range that grows/shrinks | Two indices, may move independently, not necessarily a contiguous "window" |
| Typical use | Substring/subarray with a property | Sorted-array pair search, palindrome check, partitioning |
| State tracked | Window contents (map/count/sum) | Just the two positions + comparison logic |
| Overlap | Sliding window IS a special two-pointer case | Broader category |

## HashMap vs HashSet

| | HashMap | HashSet |
|---|---|---|
| Stores | Key-value pairs | Keys only (backed internally by a HashMap) |
| Use case | Counting, lookup with associated data | Membership/duplicate check |
| Extra memory | Higher (stores values too) | Lower |

## KMP vs Rabin-Karp

| | KMP | Rabin-Karp |
|---|---|---|
| Worst case | O(n+m) guaranteed | O(n·m) with adversarial hash collisions |
| Approach | Failure function (prefix-suffix) | Rolling hash comparison |
| Multiple patterns | Awkward (needs one lps per pattern) | Natural (hash set of pattern hashes) |
| Use when | Guaranteed worst case matters | Searching many patterns at once, or approximate/plagiarism detection |

## Kadane vs Prefix Sum

| | Kadane | Prefix Sum |
|---|---|---|
| Answers | Max/min contiguous subarray sum | Any range sum query, subarray-sum-equals-K |
| Extra structure | None (O(1) space) | Prefix array or hashmap of sums |
| Updates | N/A (single pass) | Static array — use Fenwick/Segment Tree if mutable |

## QuickSort vs MergeSort

| | QuickSort | MergeSort |
|---|---|---|
| Worst case | O(n²) (bad pivot choices) | O(n log n) guaranteed |
| Space | O(log n) in-place | O(n) extra |
| Stable | No | Yes |
| Practical speed | Usually faster (cache-friendly, in-place) | Slightly slower constant factor |
| Use when | Average-case speed matters, memory constrained | Stability or worst-case guarantee required |

## Heap vs Quick Select

| | Heap | Quick Select |
|---|---|---|
| Time | O(n log k) | O(n) average, O(n²) worst |
| Space | O(k) | O(1) (in-place) |
| Streaming data | ✅ Works incrementally | ❌ Needs full array upfront |
| Guarantee | Predictable | Needs randomized pivot to avoid worst case |

## Binary Search vs Binary Search on Answer

| | Binary Search | Binary Search on Answer |
|---|---|---|
| Searches over | Actual array indices | An abstract "answer" range |
| Requires | Sorted array | Monotonic feasibility function |
| Example | Find target in sorted array | Minimize max load, Koko eating bananas |

## Trie vs HashMap

| | Trie | HashMap |
|---|---|---|
| Prefix queries | O(L), natural | Needs extra work (iterate all keys) |
| Memory | Higher (node per character) | Lower for sparse/short keys |
| Use when | Prefix search, autocomplete | Simple exact-key lookup |

## DFS vs BFS (Grid/Graph)

| | DFS | BFS |
|---|---|---|
| Finds | Any path, connectivity, components | **Shortest** path (unweighted) |
| Space | O(depth) recursion stack | O(width) queue — can be larger |
| Use when | Counting islands, exhaustive search, backtracking | Shortest steps, level-order, multi-source spread |

## Deque vs Stack

| | Deque | Stack (or `Deque` used as stack) |
|---|---|---|
| Ends usable | Both ends | One end (top) |
| Use when | Sliding window max/min, palindrome check | Next greater/smaller, parentheses matching, backtracking |

## Monotonic Queue vs Heap

| | Monotonic Queue (Deque) | Heap |
|---|---|---|
| Window operations | O(1) amortized per element | O(log k) per element |
| Use when | Fixed sliding window max/min | Arbitrary insert/extract-min without window structure |
| Removability | Only from window boundary (front) | Any element (with lazy deletion trick) |

## Fenwick Tree vs Segment Tree

| | Fenwick Tree | Segment Tree |
|---|---|---|
| Supports | Prefix sum / point update (mainly) | Any associative op: min, max, gcd, sum |
| Code complexity | Simple (~15 lines) | More complex (~40+ lines), lazy propagation for range updates |
| Space | O(n) | O(4n) typical |
| Use when | Just need sums | Need min/max/range-updates |

## Union Find vs BFS/DFS (Connectivity)

| | Union Find | BFS/DFS |
|---|---|---|
| Dynamic edges (added over time) | ✅ Excellent (near O(1) per op) | ❌ Needs full re-traversal |
| Static graph, one-time query | Overkill | ✅ Simpler, sufficient |
| Use when | "Redundant connection", incremental merges | "Number of islands" (one-shot grid scan) |

Next: [[13-Complexity-Cheat-Sheet]]
