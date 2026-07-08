---
tags: [dsa, pattern-recognition]
---

# 2. Interview Pattern Recognition

⬅ [[00-Home]] | Prev: [[01-Mind-Map]] | Next: [[03-Arrays-Foundations]]

> [!tip] How to use
> When you read a problem, scan the **Clue** column first. That's your fastest route to the right pattern.

| # | Question Clue | Pattern | Time Complexity | Example Problems |
|---|---|---|---|---|
| 1 | "subarray sum equals K" | [[03-Arrays-Foundations#Prefix Sum\|Prefix Sum]] + HashMap | O(n) | Subarray Sum Equals K (560) |
| 2 | "longest substring without repeating" | [[10-Strings-Foundations#Sliding Window (Strings)\|Sliding Window]] | O(n) | LC 3 |
| 3 | "sorted array, find pair/target" | [[03-Arrays-Foundations#Two Pointer\|Two Pointer]] | O(n) | Two Sum II (167) |
| 4 | "k closest points/elements" | [[07-Arrays-Stack-Queue-Heap#Heap\|Heap]] | O(n log k) | K Closest Points (973) |
| 5 | "array rotation" | Reversal Algorithm | O(n) | Rotate Array (189) |
| 6 | "majority element (> n/2)" | Boyer-Moore Voting | O(n) | Majority Element (169) |
| 7 | "contains duplicates" | Hashing | O(n) | Contains Duplicate (217) |
| 8 | "maximum contiguous sum" | [[03-Arrays-Foundations#Kadane's Algorithm\|Kadane]] | O(n) | Max Subarray (53) |
| 9 | "range sum queries, immutable" | Prefix Sum | O(1) query | Range Sum Query (303) |
| 10 | "range update queries" | Difference Array / Fenwick | O(log n) update | Range Add Query |
| 11 | "find peak element" | Binary Search | O(log n) | Find Peak Element (162) |
| 12 | "search in rotated sorted array" | Modified Binary Search | O(log n) | LC 33 |
| 13 | "minimize the maximum / maximize the minimum" | Binary Search on Answer | O(n log(max-min)) | Split Array Largest Sum (410) |
| 14 | "sort colors / 0s 1s 2s" | Dutch National Flag | O(n) | Sort Colors (75) |
| 15 | "merge overlapping intervals" | Merge Intervals | O(n log n) | LC 56 |
| 16 | "meeting rooms / min resources over time" | Sweep Line | O(n log n) | Meeting Rooms II (253) |
| 17 | "numbers in range [1,n], find missing/duplicate" | Cyclic Sort | O(n) | Find Missing Number (268) |
| 18 | "kth largest/smallest element" | Quick Select / Heap | O(n) avg | Kth Largest (215) |
| 19 | "anagram grouping" | Hashing (sorted key / count key) | O(n·k log k) | Group Anagrams (49) |
| 20 | "valid anagram" | Frequency Array | O(n) | Valid Anagram (242) |
| 21 | "next greater/smaller element" | Monotonic Stack | O(n) | Next Greater Element (496) |
| 22 | "sliding window maximum" | Monotonic Deque | O(n) | LC 239 |
| 23 | "top K frequent elements" | Heap / Bucket Sort | O(n log k) or O(n) | Top K Frequent (347) |
| 24 | "trapping rain water" | Two Pointer / Monotonic Stack | O(n) | LC 42 |
| 25 | "container with most water" | Two Pointer | O(n) | LC 11 |
| 26 | "longest palindromic substring" | Expand Around Center / Manacher | O(n²) or O(n) | LC 5 |
| 27 | "check if string is palindrome" | Two Pointer | O(n) | Valid Palindrome (125) |
| 28 | "pattern search in text" | KMP / Rabin-Karp / Z-Algo | O(n+m) | Implement strStr (28) |
| 29 | "longest common prefix" | Vertical scan / Trie | O(n·m) | LC 14 |
| 30 | "word search / autocomplete" | Trie | O(word length) | Implement Trie (208) |
| 31 | "spiral order traversal" | Matrix boundary simulation | O(n·m) | Spiral Matrix (54) |
| 32 | "rotate image/matrix 90°" | Transpose + Reverse | O(n²) | Rotate Image (48) |
| 33 | "number of islands / connected regions" | BFS/DFS Grid or Union Find | O(n·m) | Number of Islands (200) |
| 34 | "flood fill / paint bucket" | DFS/BFS Flood Fill | O(n·m) | Flood Fill (733) |
| 35 | "minimum window substring" | Sliding Window Variable | O(n) | LC 76 |
| 36 | "longest subarray with at most K distinct" | Sliding Window Variable + HashMap | O(n) | LC 340 |
| 37 | "median of data stream" | Two Heaps | O(log n) per insert | Find Median from Data Stream (295) |
| 38 | "kth smallest in sorted matrix" | Heap / Binary Search on Answer | O(k log n) | LC 378 |
| 39 | "candy distribution / minimum increments" | Greedy | O(n) | Candy (135) |
| 40 | "jump game / reachability" | Greedy | O(n) | Jump Game (55) |
| 41 | "gas station circular tour" | Greedy | O(n) | Gas Station (134) |
| 42 | "product of array except self" | Prefix × Suffix product | O(n) | LC 238 |
| 43 | "count inversions" | Merge Sort / Fenwick Tree | O(n log n) | Count Inversions |
| 44 | "longest increasing subsequence length" | Binary Search + Patience Sorting | O(n log n) | LC 300 |
| 45 | "distinct substrings count" | Suffix Array / Rolling Hash | O(n log n) | — |
| 46 | "repeated substring pattern" | KMP failure function | O(n) | LC 459 |
| 47 | "minimum swaps / minimum operations to sort" | Cyclic Sort / Greedy | O(n) | Minimum Swaps to Sort |
| 48 | "circular array problems" | Two Pointer / Modulo indexing | O(n) | Circular Array Loop (457) |

> [!note] Rule of thumb
> If you see **"contiguous" + "sum"** → think Prefix Sum / Kadane first.
> If you see **"substring/subarray" + "condition that grows/shrinks"** → think Sliding Window.
> If you see **"kth" / "top-k" / "closest-k"** → think Heap or Quick Select.
> If you see **"next greater/smaller"** → think Monotonic Stack.

Next: [[03-Arrays-Foundations]]
