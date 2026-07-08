---
tags: [dsa, arrays, intervals, sorting, selection]
---

# 5. Arrays — Merge Intervals, Sweep Line, Cyclic Sort, Quick Select, Moore Voting

⬅ [[00-Home]] | Prev: [[04-Arrays-Windows-Search]] | Next: [[06-Arrays-Hashing-Greedy]]

---

## Merge Intervals

> [!tip] When to use
> "Merge overlapping intervals", "insert interval", "can attend all meetings".

**Intuition:** Sort by start time; walk through, merging whenever the current interval's start ≤ previous interval's end.

**Visual:**
```
[[1,3],[2,6],[8,10],[15,18]]  (sorted by start)
[1,3] & [2,6] overlap (2<=3) -> merge to [1,6]
[1,6] & [8,10] no overlap -> keep separate
Result: [[1,6],[8,10],[15,18]]
```

**Complexity:** Time O(n log n) (sort) · Space O(n)

**Template:**
```java
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
List<int[]> merged = new ArrayList<>();
for (int[] cur : intervals) {
    if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < cur[0]) {
        merged.add(cur);
    } else {
        merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], cur[1]);
    }
}
```

**Common Bugs:** forgetting to sort first; using `<` instead of `<=` for the overlap boundary (matters if touching intervals should merge).

**Edge Cases:** empty input, single interval, all intervals overlap into one, no overlaps at all.

**Related:** [[05-Arrays-Sorting-Selection#Sweep Line|Sweep Line]]

**Must Solve:** Medium: Merge Intervals (56), Insert Interval (57), Meeting Rooms (252) · Hard: —

---

## Sweep Line

> [!tip] When to use
> "Minimum meeting rooms", "max overlapping intervals at any point in time".

**Intuition:** Convert each interval into two events (+1 at start, -1 at end); sort all events by time and sweep, tracking a running counter — its max is the answer.

**Visual:**
```
Intervals: [0,30],[5,10],[15,20]
Events (time,delta): (0,+1)(5,+1)(10,-1)(15,+1)(20,-1)(30,-1)
Sweep: 1, 2, 1, 2, 1, 0  -> max concurrent = 2
```

**Complexity:** Time O(n log n) · Space O(n)

**Template:**
```java
int[][] events = new int[2 * n][2];
for (int i = 0; i < n; i++) {
    events[2*i]   = new int[]{intervals[i][0], 1};
    events[2*i+1] = new int[]{intervals[i][1], -1};
}
Arrays.sort(events, (a, b) -> a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]); // end-before-start on ties if needed
int running = 0, maxRooms = 0;
for (int[] e : events) { running += e[1]; maxRooms = Math.max(maxRooms, running); }
```

**Common Bugs:** wrong tie-break order when a meeting ends exactly when another starts (decide if that counts as overlap or not, per problem statement).

**Edge Cases:** all meetings identical, no meetings, one giant meeting spanning all others.

**Related:** [[05-Arrays-Sorting-Selection#Merge Intervals|Merge Intervals]]

**Must Solve:** Medium: Meeting Rooms II (253), Car Pooling (1094) · Hard: My Calendar III (732)

---

## Cyclic Sort

> [!tip] When to use
> Array contains numbers in range **[1, n]** or **[0, n-1]** — find missing/duplicate/first missing positive.

**Intuition:** Place each number at its "correct" index (`value` at index `value-1`) via swaps; then scan for the first mismatch.

**Visual:**
```
[3,1,5,4,2] (values 1..5, indices 0..4)
i=0: arr[0]=3, should be at index 2 -> swap -> [5,1,3,4,2]
i=0: arr[0]=5, should be at index 4 -> swap -> [2,1,3,4,5]
i=0: arr[0]=2, should be at index 1 -> swap -> [1,2,3,4,5]
i=0: arr[0]=1 correct, move on... already sorted!
```

**Complexity:** Time O(n) · Space O(1)

**Template:**
```java
int i = 0;
while (i < arr.length) {
    int correct = arr[i] - 1; // for 1..n values
    if (arr[i] != arr[correct]) {
        swap(arr, i, correct);
    } else {
        i++;
    }
}
// second pass: find first index where arr[i] != i+1 -> that's the missing number
```

**Common Bugs:** infinite loop if you `i++` after a swap instead of re-checking the same index; not handling duplicate values (causes infinite swap loop — add an equality guard).

**Edge Cases:** all values already in place, all duplicates of one value, missing multiple numbers.

**Related:** [[06-Arrays-Hashing-Greedy#HashSet|HashSet]] (alternative O(n) space approach)

**Must Solve:** Easy: Missing Number (268) · Medium: Find All Duplicates (442), First Missing Positive (41 — Hard actually) · Hard: First Missing Positive (41)

---

## Quick Select

> [!tip] When to use
> "Kth largest/smallest element" without needing a fully sorted array.

**Intuition:** Like QuickSort's partition step, but recurse into only the side that contains the kth index — average O(n) instead of O(n log n).

**Visual:**
```
Find 3rd largest in [3,2,1,5,6,4] -> kth smallest index = n-k = 3
Partition around pivot, check pivot's final index vs target index,
recurse into left or right half only.
```

**Complexity:** Time O(n) average, O(n²) worst case · Space O(1) (in-place) / O(log n) (recursion stack)

**Template:**
```java
int quickSelect(int[] arr, int lo, int hi, int k) { // k = 0-indexed target position
    int pivotIndex = partition(arr, lo, hi);
    if (pivotIndex == k) return arr[pivotIndex];
    else if (pivotIndex < k) return quickSelect(arr, pivotIndex + 1, hi, k);
    else return quickSelect(arr, lo, pivotIndex - 1, k);
}

int partition(int[] arr, int lo, int hi) {
    int pivot = arr[hi], i = lo;
    for (int j = lo; j < hi; j++) {
        if (arr[j] < pivot) swap(arr, i++, j);
    }
    swap(arr, i, hi);
    return i;
}
```

**Common Bugs:** off-by-one converting "kth largest" to a 0-indexed target; forgetting to randomize pivot (worst-case O(n²) on adversarial/sorted input — use random pivot or median-of-three).

**Edge Cases:** k = 1 or k = n, duplicate values, already sorted array (worst case without randomized pivot).

**Related:** [[07-Arrays-Stack-Queue-Heap#Heap|Heap]] (O(n log k) alternative, simpler to reason about)

**Must Solve:** Medium: Kth Largest Element in an Array (215) · Hard: —

---

## Moore's Voting Algorithm

> [!tip] When to use
> "Majority element" — appears **more than ⌊n/2⌋** times.

**Intuition:** Maintain a candidate and a counter. Increment when you see the candidate, decrement otherwise; when the counter hits 0, switch candidates. The true majority always survives.

**Visual:**
```
[2,2,1,1,1,2,2]
cand=2 cnt=1
cand=2 cnt=2
1: cnt=1
1: cnt=0 -> switch cand=1, cnt=1
1: cnt=2
2: cnt=1
2: cnt=0 -> switch cand=2... final candidate=2 (need verification pass if not guaranteed majority exists)
```

**Complexity:** Time O(n) · Space O(1)

**Template:**
```java
int candidate = 0, count = 0;
for (int x : arr) {
    if (count == 0) candidate = x;
    count += (x == candidate) ? 1 : -1;
}
// If majority isn't guaranteed to exist, do a second pass to verify count(candidate) > n/2
```

**Common Bugs:** skipping the verification pass when a majority element isn't guaranteed to exist.

**Edge Cases:** no true majority (verify!), all elements identical, exactly n/2 (not majority — needs `>` not `>=`).

**Variations:** Majority Element II (find elements appearing > n/3 times — generalizes to tracking 2 candidates).

**Related:** [[06-Arrays-Hashing-Greedy#HashMap|HashMap]] (simpler O(n) space alternative)

**Must Solve:** Easy: Majority Element (169) · Medium: Majority Element II (229) · Hard: —

---

## Sorting-Based Approaches (Overview)

> [!tip] When to use
> When the problem becomes trivial once sorted (dedup, closest pair, scheduling by one attribute).

**Template — Java's built-in sort:**
```java
Arrays.sort(arr);                                  // primitives, dual-pivot quicksort, O(n log n) avg
Arrays.sort(objArr, Comparator.comparing(Obj::getKey)); // objects, TimSort, STABLE, O(n log n) guaranteed
```

> [!warning] Key fact
> `Arrays.sort(int[])` uses **Dual-Pivot QuickSort** (NOT stable, O(n log n) avg but O(n²) worst case theoretically).
> `Arrays.sort(Object[])` uses **TimSort** (stable, guaranteed O(n log n)).

**Common Bugs:** assuming primitive array sort is stable (it isn't — doesn't matter for primitives, but relevant if you pack extra info into bits).

**Related:** [[07-Arrays-Stack-Queue-Heap]] (Merge Sort, Quick Sort, Bucket Sort, Counting Sort details)

**Must Solve:** Easy: Merge Sorted Array (88) · Medium: Sort Array by Parity (905) · Hard: Count of Smaller Numbers After Self (315)

Next: [[06-Arrays-Hashing-Greedy]]
