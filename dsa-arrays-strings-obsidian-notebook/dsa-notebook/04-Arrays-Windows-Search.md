---
tags: [dsa, arrays, sliding-window, binary-search]
---

# 4. Arrays — Sliding Window & Binary Search

⬅ [[00-Home]] | Prev: [[03-Arrays-Foundations]] | Next: [[05-Arrays-Sorting-Selection]]

---

## Sliding Window Fixed

> [!tip] When to use
> Window size **K is given** — max/min/average of every window of size K.

**Intuition:** Slide a fixed-size window; add the new element entering, remove the one leaving. Avoids recomputation.

**Visual:**
```
arr: [1,3,-1,-3,5,3,6,7], k=3
[1,3,-1] -3 5 3 6 7   -> max=3
1 [3,-1,-3] 5 3 6 7   -> max=3
1 3 [-1,-3,5] 3 6 7   -> max=5
```

**Complexity:** Time O(n) · Space O(1) (or O(k) if using a deque, see [[07-Arrays-Stack-Queue-Heap]])

**Template:**
```java
int windowSum = 0;
for (int i = 0; i < k; i++) windowSum += arr[i];
int best = windowSum;
for (int i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];
    best = Math.max(best, windowSum);
}
```

**Common Bugs:** forgetting to initialize the first window before sliding; off-by-one on `arr[i-k]`.

**Edge Cases:** k > array length, k == array length, k == 1.

**Related:** [[07-Arrays-Stack-Queue-Heap#Monotonic Queue|Monotonic Queue]] (for sliding window max/min in O(n))

**Must Solve:** Easy: Maximum Average Subarray I (643) · Medium: Sliding Window Maximum (239 — needs deque) · Hard: —

---

## Sliding Window Variable

> [!tip] When to use
> "Longest/shortest subarray/substring satisfying a condition" — window **grows and shrinks**.

**Intuition:** Expand right pointer to grow window; when the condition breaks, shrink from the left until valid again.

**Visual:**
```
"Longest substring with at most 2 distinct chars": s = "eceba"
R=0 e         window={e}          valid
R=1 ec        window={e,c}        valid
R=2 ece       window={e,c}        valid (e repeats)
R=3 eceb      window={e,c,b} -> invalid (3 distinct) -> shrink L until valid
     "ceb"    window={c,e,b} still 3 -> shrink more -> "eb" valid
```

**Complexity:** Time O(n) — each pointer moves at most n times · Space O(k) for the window's tracking structure

**Template:**
```java
int left = 0, best = 0;
Map<Character, Integer> count = new HashMap<>();
for (int right = 0; right < s.length(); right++) {
    count.merge(s.charAt(right), 1, Integer::sum);
    while (/* window invalid, e.g. count.size() > k */ false) {
        char c = s.charAt(left);
        count.put(c, count.get(c) - 1);
        if (count.get(c) == 0) count.remove(c);
        left++;
    }
    best = Math.max(best, right - left + 1);
}
```

**Common Bugs:** shrinking with `if` instead of `while`; forgetting to remove a key when its count hits 0 (breaks `.size()` checks); not updating `best` at the right point (before or after shrink, depending on problem).

**Edge Cases:** entire string/array is the answer, no valid window exists (return 0), window of size 1.

**Variations:** minimum window substring (shrink to find *smallest* valid window instead of largest), at-most-K distinct, exactly-K distinct (= atMostK(K) − atMostK(K−1) trick).

**Related:** [[03-Arrays-Foundations#Two Pointer|Two Pointer]] (same-direction two pointer is the mechanism)

**Must Solve:**
- Easy: —
- Medium: Longest Substring Without Repeating Characters (3), Longest Substring with At Most K Distinct (340), Fruit Into Baskets (904)
- Hard: Minimum Window Substring (76)

---

## Binary Search

> [!tip] When to use
> Sorted array, "find target/insertion point", O(log n) required.

**Intuition:** Repeatedly halve the search space by comparing the middle element to the target.

**Visual:**
```
[1,3,5,7,9,11], target=7
lo=0 hi=5 mid=2 (5) < 7 -> lo=3
lo=3 hi=5 mid=4 (9) > 7 -> hi=3
lo=3 hi=3 mid=3 (7) == 7 -> FOUND
```

**Complexity:** Time O(log n) · Space O(1)

**Template (classic):**
```java
int lo = 0, hi = arr.length - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;   // avoids overflow!
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```

**Lower Bound (first index ≥ target):**
```java
int lo = 0, hi = arr.length;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid;
}
return lo;
```

**Upper Bound (first index > target):**
```java
int lo = 0, hi = arr.length;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] <= target) lo = mid + 1;
    else hi = mid;
}
return lo;
```

**Common Bugs:** `mid = (lo + hi) / 2` → integer overflow for large indices; `lo <= hi` vs `lo < hi` mismatch causing infinite loop; not moving `lo`/`hi` past `mid` (infinite loop).

**Edge Cases:** empty array, target smaller than all/larger than all elements, duplicates (lower/upper bound needed), array of size 1.

**Variations:** search in rotated sorted array (compare halves to find the sorted side first), search in 2D matrix, find peak element, first/last occurrence.

**Related:** [[04-Arrays-Windows-Search#Binary Search on Answer|Binary Search on Answer]]

**Must Solve:**
- Easy: Binary Search (704), First Bad Version (278)
- Medium: Search in Rotated Sorted Array (33), Find First and Last Position (34)
- Hard: Median of Two Sorted Arrays (4)

---

## Binary Search on Answer

> [!tip] When to use
> "Minimize the maximum" / "maximize the minimum" over a **monotonic feasibility function** — the answer itself is what you binary search on, not an index.

**Intuition:** If `canAchieve(x)` is monotonic (true for all x ≥ threshold, false below, or vice versa), binary search the smallest/largest `x` where it flips.

**Visual:**
```
Split array to minimize largest sum among k parts.
Binary search on "max allowed sum" from max(arr) to sum(arr).
feasible(x) = can we split into <= k parts each with sum <= x?
   x too small -> needs too many parts -> infeasible
   x too large -> trivially feasible
   Find the SMALLEST feasible x.
```

**Complexity:** Time O(n log(range)) where range = hi - lo of the answer space · Space O(1)

**Template:**
```java
int lo = minPossibleAnswer, hi = maxPossibleAnswer;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) hi = mid;      // mid works, try smaller
    else lo = mid + 1;                // mid too small, need bigger
}
return lo;

boolean feasible(int x) {
    // problem-specific monotonic check, e.g. count parts needed with cap x
    return true;
}
```

**Common Bugs:** `feasible()` not actually monotonic (invalid binary search); wrong initial `lo`/`hi` bounds; off-by-one converging to wrong side.

**Edge Cases:** single element array, k = 1 (whole array is one part), k = n (each element its own part).

**Related:** [[04-Arrays-Windows-Search#Binary Search|Binary Search]] (same mechanics, different search space)

**Must Solve:**
- Medium: Koko Eating Bananas (875), Capacity To Ship Packages Within D Days (1011)
- Hard: Split Array Largest Sum (410)

---

## Dutch National Flag

> [!tip] When to use
> Sort an array of **3 distinct values** (e.g., 0/1/2) in-place, single pass.

**Intuition:** Maintain three regions using three pointers: `low` (0s boundary), `mid` (current), `high` (2s boundary). Swap elements into their correct region as you scan.

**Visual:**
```
[2,0,2,1,1,0], low=0 mid=0 high=5
mid=2 -> swap(low,mid), low++, mid++
... continues until mid > high
Final: [0,0,1,1,2,2]
```

**Complexity:** Time O(n) · Space O(1)

**Template:**
```java
int low = 0, mid = 0, high = arr.length - 1;
while (mid <= high) {
    if (arr[mid] == 0) { swap(arr, low++, mid++); }
    else if (arr[mid] == 1) { mid++; }
    else { swap(arr, mid, high--); }   // don't increment mid here!
}
```

**Common Bugs:** incrementing `mid` after swapping with `high` (the swapped-in value hasn't been checked yet!); wrong pointer initialization.

**Edge Cases:** all same value, already sorted, single element.

**Variations:** generalize to k distinct values (counting sort is simpler for k > 3).

**Related:** [[07-Arrays-Stack-Queue-Heap#Counting Sort|Counting Sort]]

**Must Solve:** Medium: Sort Colors (75) · Hard: —

Next: [[05-Arrays-Sorting-Selection]]
