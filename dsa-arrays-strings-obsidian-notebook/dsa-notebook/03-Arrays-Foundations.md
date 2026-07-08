---
tags: [dsa, arrays, prefix-sum, kadane, two-pointer]
---

# 3. Arrays — Foundations

⬅ [[00-Home]] | Prev: [[02-Pattern-Recognition]] | Next: [[04-Arrays-Windows-Search]]

---

## Basic Traversal

> [!tip] When to use
> Any single-pass computation: min/max, sum, counting, simple validation.

**Intuition:** Visit each element once, carry a running result.

**Template:**
```java
int result = init;
for (int i = 0; i < arr.length; i++) {
    // process arr[i]
}
```

**Complexity:** Time O(n) · Space O(1)

**Common Bugs:** off-by-one on `<` vs `<=`; forgetting empty-array check.

**Edge Cases:** empty array, single element, all same values.

**Related:** [[03-Arrays-Foundations#Prefix Sum|Prefix Sum]], [[03-Arrays-Foundations#Two Pointer|Two Pointer]]

**Must Solve:** Easy: Find Max (LC 1750-style basics) · Medium: Best Time to Buy/Sell Stock (121) · Hard: —

---

## Prefix Sum

> [!tip] When to use
> Repeated **range sum** queries, "subarray sum equals K", equilibrium index, product except self (variant).

**Intuition:** Precompute cumulative sums so any range sum = `prefix[j] - prefix[i-1]` in O(1).

**Visual:**
```
arr:      3   1   4   1   5
prefix: 0 3   4   8   9  14
sum(1..3) = prefix[4] - prefix[1] = 9 - 3 = 6  → arr[1]+arr[2]+arr[3] = 1+4+1 = 6 ✅
```

**Dry Run (Subarray Sum = K):** arr=[1,2,3], k=3 → running sum 1,3,6; map{0:1,1:1,3:1}; at sum=3 check map.get(3-3=0)=1 → count=1; at sum=6 check map.get(3)=1 → count=2. Answer: 2.

**Complexity:** Build O(n) · Query O(1) · Space O(n)

**Advantages:** O(1) range queries after O(n) preprocessing.
**Limitations:** Rebuilding on updates is O(n) — use Fenwick/Segment Tree for frequent updates ([[08-Arrays-Advanced-DS]]).

**Template:**
```java
int[] prefix = new int[n + 1];
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + arr[i];
// range sum [l, r] inclusive:
int rangeSum = prefix[r + 1] - prefix[l];

// Subarray sum equals K (HashMap of prefix sums)
Map<Integer, Integer> seen = new HashMap<>();
seen.put(0, 1);
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    count += seen.getOrDefault(sum - k, 0);
    seen.merge(sum, 1, Integer::sum);
}
```

**Common Bugs:** forgetting `seen.put(0,1)` seed; off-by-one with `prefix[i+1]` vs `prefix[i]`.

**Edge Cases:** negative numbers (sums can repeat/decrease), k = 0, empty array.

**Variations:** 2D prefix sum (matrix range sum), prefix XOR, prefix product (careful with zeros).

**Related:** [[03-Arrays-Foundations#Kadane's Algorithm|Kadane]], [[08-Arrays-Advanced-DS#Fenwick Tree (Binary Indexed Tree)|Fenwick Tree]]

**Must Solve:**
- Easy: Range Sum Query - Immutable (303)
- Medium: Subarray Sum Equals K (560), Product of Array Except Self (238)
- Hard: Count of Range Sum (327)

---

## Difference Array

> [!tip] When to use
> Multiple **range update** operations, apply all at once, then read final array.

**Intuition:** Store deltas at boundaries; a single sweep (prefix sum of diffs) reconstructs the final array.

**Visual:**
```
Apply +3 to range [1,3] on array of size 5:
diff:    [0, +3, 0, 0, -3, 0]
prefix of diff → [0, 3, 3, 3, 0]  = final added amounts
```

**Template:**
```java
int[] diff = new int[n + 1];
void addRange(int l, int r, int val) {
    diff[l] += val;
    diff[r + 1] -= val;
}
int[] build() {
    int[] result = new int[n];
    int running = 0;
    for (int i = 0; i < n; i++) {
        running += diff[i];
        result[i] = running;
    }
    return result;
}
```

**Complexity:** Each update O(1) · Build O(n) · Space O(n)

**Common Bugs:** forgetting `r+1` bound check (array overflow), applying updates one at a time instead of batching.

**Edge Cases:** overlapping ranges, range covering entire array, single-index range (l == r).

**Related:** [[03-Arrays-Foundations#Prefix Sum|Prefix Sum]] (inverse operation)

**Must Solve:** Medium: Range Addition (370), Corporate Flight Bookings (1109) · Hard: Car Pooling (1094)

---

## Kadane's Algorithm

> [!tip] When to use
> "Maximum (or minimum) contiguous subarray sum" — the moment you see **"contiguous" + "max/min sum"**, think Kadane.

**Intuition:** At each index, decide: extend the previous subarray or start fresh. Keep a running "best ending here" and a global best.

**Visual:**
```
arr: -2  1  -3  4  -1  2  1  -5  4
cur:  -2 1  -2  4   3  5  6   1  5
best: -2 1   1  4   4  5  6   6  6
```

**Dry Run:** as above → answer = 6 (subarray [4,-1,2,1]).

**Complexity:** Time O(n) · Space O(1)

**Advantages:** Single pass, constant space, elegant.
**Limitations:** Only works for contiguous subarrays; doesn't directly give the subarray indices unless tracked separately.

**Template:**
```java
int maxSoFar = arr[0], maxEndingHere = arr[0];
for (int i = 1; i < arr.length; i++) {
    maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
    maxSoFar = Math.max(maxSoFar, maxEndingHere);
}
return maxSoFar;
```

**Common Bugs:** initializing `maxSoFar = 0` (fails on all-negative arrays); resetting `maxEndingHere` to 0 instead of comparing.

**Edge Cases:** all negative numbers, single element, all zeros.

**Variations:** Maximum product subarray (track both max & min running product, due to sign flips), circular array Kadane (max of normal Kadane vs total - min-subarray), 2D Kadane (max submatrix sum).

**Related:** [[03-Arrays-Foundations#Prefix Sum|Prefix Sum]] (max subarray = max(prefix[j]) - min(prefix[i]))

**Must Solve:**
- Easy: —
- Medium: Maximum Subarray (53), Maximum Product Subarray (152)
- Hard: Maximum Sum Circular Subarray (918)

---

## Two Pointer

> [!tip] When to use
> Sorted array + pair/triplet target sum, removing duplicates in-place, reversing, palindrome check, merging two sorted arrays.

**Intuition:** Two indices move toward each other (or in the same direction) based on a comparison, avoiding a nested loop.

**Visual:**
```
sorted: [2, 7, 11, 15], target = 9
L=0(2)                R=3(15)  sum=17 > 9 → R--
L=0(2)          R=2(11)        sum=13 > 9 → R--
L=0(2)     R=1(7)               sum=9  == 9 → FOUND
```

**Complexity:** Time O(n) (vs O(n²) brute force) · Space O(1)

**Template:**
```java
int l = 0, r = arr.length - 1;
while (l < r) {
    int sum = arr[l] + arr[r];
    if (sum == target) { /* found */ break; }
    else if (sum < target) l++;
    else r--;
}
```

**Common Bugs:** using on unsorted array without sorting first; `l <= r` vs `l < r` mismatch; not skipping duplicates in 3Sum-style problems.

**Edge Cases:** empty array, single element, all duplicates, no valid pair exists.

**Variations:** Same-direction two pointer (fast/slow for cycle detection, remove duplicates in-place), 3Sum / 4Sum (fix one/two pointers, two-pointer the rest), trapping rain water (two pointers from both ends tracking max height).

**Related:** [[04-Arrays-Windows-Search#Sliding Window Variable|Sliding Window]] (same-direction two pointer is a special case)

**Must Solve:**
- Easy: Valid Palindrome (125), Two Sum II (167)
- Medium: 3Sum (15), Container With Most Water (11)
- Hard: Trapping Rain Water (42)

---

Next: [[04-Arrays-Windows-Search]]
