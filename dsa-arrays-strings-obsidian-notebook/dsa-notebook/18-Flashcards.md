---
tags: [dsa, flashcards]
---

# 18. Flashcards (150+)

⬅ [[00-Home]] | Prev: [[17-Interview-Flowchart]] | Next: [[19-Common-Mistakes]]

> [!tip] How to use
> Read the question, answer out loud, then expand to check. Don't peek first!

## Arrays Foundations

<details><summary>Q: When should Kadane's Algorithm be used?</summary>A: When you need the maximum (or minimum) sum of a <b>contiguous</b> subarray.</details>

<details><summary>Q: What's the key initialization mistake in Kadane's?</summary>A: Initializing the running/best sum to 0 instead of the first element — fails on all-negative arrays.</details>

<details><summary>Q: What does Prefix Sum let you compute in O(1)?</summary>A: The sum of any subrange, after O(n) preprocessing.</details>

<details><summary>Q: Why do we seed the prefix-sum HashMap with {0: 1}?</summary>A: To correctly count subarrays that start at index 0 and sum exactly to K.</details>

<details><summary>Q: When is a Difference Array preferred over Prefix Sum?</summary>A: When you have many <b>range update</b> operations to apply before reading the final array.</details>

<details><summary>Q: What is the core idea of Two Pointer on a sorted array?</summary>A: Move one pointer from each end (or both forward) based on a sum/comparison to avoid nested loops.</details>

<details><summary>Q: Two Pointer requires the array to be what?</summary>A: Sorted (or sortable without violating constraints).</details>

<details><summary>Q: Sliding Window works when the window property is what?</summary>A: Maintainable incrementally — you can update the window state in O(1) as elements enter/exit.</details>

<details><summary>Q: Fixed vs Variable Sliding Window — what's the difference?</summary>A: Fixed: window size K is given upfront. Variable: window grows/shrinks based on a condition.</details>

<details><summary>Q: What's the classic bug in Binary Search's mid calculation?</summary>A: `(lo + hi) / 2` can overflow for large indices — use `lo + (hi - lo) / 2`.</details>

## Binary Search Family

<details><summary>Q: What does "lower bound" mean in binary search?</summary>A: The first index where `arr[i] >= target`.</details>

<details><summary>Q: What does "upper bound" mean in binary search?</summary>A: The first index where `arr[i] > target`.</details>

<details><summary>Q: What is "Binary Search on Answer"?</summary>A: Binary searching over a range of possible <i>answers</i> using a monotonic feasibility check, not over array indices.</details>

<details><summary>Q: What must be true for Binary Search on Answer to work?</summary>A: The feasibility function must be monotonic (true for all x ≥ threshold, or vice versa).</details>

<details><summary>Q: How do you search a rotated sorted array?</summary>A: Determine which half is properly sorted, check if target lies in that half's range, recurse/iterate accordingly.</details>

## Sorting & Selection

<details><summary>Q: When to use Dutch National Flag?</summary>A: Sorting an array of exactly 3 distinct values in-place, single pass.</details>

<details><summary>Q: What are the 3 pointers in Dutch National Flag?</summary>A: `low` (0s boundary), `mid` (scanner), `high` (2s boundary).</details>

<details><summary>Q: Why don't we increment `mid` after a swap with `high`?</summary>A: The swapped-in value hasn't been checked yet — must re-examine it.</details>

<details><summary>Q: When to use Cyclic Sort?</summary>A: Values are confined to range [1,n] or [0,n-1] — find missing/duplicate numbers in O(n) time, O(1) space.</details>

<details><summary>Q: Quick Select time complexity, average and worst?</summary>A: O(n) average, O(n²) worst case (mitigated with randomized pivot).</details>

<details><summary>Q: When does Moore's Voting Algorithm need a second pass?</summary>A: When a true majority element (> n/2 occurrences) isn't guaranteed to exist — verify the candidate.</details>

<details><summary>Q: Is Java's `Arrays.sort(int[])` stable?</summary>A: Doesn't matter for primitives (no associated data), but the algorithm itself (Dual-Pivot QuickSort) is not stable.</details>

<details><summary>Q: Is Java's `Arrays.sort(Object[])` stable?</summary>A: Yes — it uses TimSort, which is stable and guarantees O(n log n).</details>

<details><summary>Q: Merge Sort vs Quick Sort — which is stable?</summary>A: Merge Sort is stable; Quick Sort is not.</details>

<details><summary>Q: What causes Quick Sort's worst-case O(n²)?</summary>A: Consistently bad pivot choices (e.g., always picking the last element on already-sorted input).</details>

<details><summary>Q: When to use Bucket Sort?</summary>A: Values fall into a known bounded range — great for frequency-based top-K problems.</details>

<details><summary>Q: When to use Counting Sort?</summary>A: Small, known integer range — O(n+k) sort without comparisons.</details>

## Hashing & Greedy

<details><summary>Q: HashMap vs HashSet — key difference?</summary>A: HashMap stores key-value pairs; HashSet stores keys only (membership check).</details>

<details><summary>Q: Why check `map.containsKey(complement)` before inserting in Two Sum?</summary>A: To avoid using the same element twice as its own complement.</details>

<details><summary>Q: When is a Frequency Array preferred over a HashMap?</summary>A: When the key range is small and known (e.g., lowercase letters) — faster, no hashing/boxing overhead.</details>

<details><summary>Q: What makes a Greedy algorithm valid?</summary>A: Provable local-optimal-implies-global-optimal structure (exchange argument or matroid property) — not just "it feels right."</details>

<details><summary>Q: Jump Game greedy — what do you track?</summary>A: The maximum index reachable so far (`maxReach`), updated as you scan left to right.</details>

Next Flashcards: [[18b-Flashcards-Continued]]
