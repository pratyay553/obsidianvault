---
tags: [dsa, arrays, monotonic-stack, heap, sorting]
---

# 7. Arrays — Monotonic Stack/Queue, Heap, Sorting Algorithms

⬅ [[00-Home]] | Prev: [[06-Arrays-Hashing-Greedy]] | Next: [[08-Arrays-Advanced-DS]]

---

## Monotonic Stack

> [!tip] When to use
> "Next greater/smaller element", "daily temperatures", "largest rectangle in histogram".

**Intuition:** Keep the stack in increasing (or decreasing) order; pop elements that violate the order as you scan, connecting each popped element to the current one as its "answer".

**Visual (Next Greater Element):**
```
[2,1,2,4,3], stack keeps indices, decreasing values
i=0(2): stack=[0]
i=1(1): 1<2, stack=[0,1]
i=2(2): 2>=arr[1]=1 -> pop 1, ans[1]=2; 2>=arr[0]=2? no(equal doesn't pop if strict >) stack=[0,2]
i=3(4): pop 2(2<4) ans[2]=4; pop 0(2<4) ans[0]=4; stack=[3]
i=4(3): 3<4, stack=[3,4]
remaining stack -> no next greater -> -1
```

**Complexity:** Time O(n) — each element pushed/popped once · Space O(n)

**Template:**
```java
int[] result = new int[n];
Arrays.fill(result, -1);
Deque<Integer> stack = new ArrayDeque<>(); // stores indices
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
        result[stack.pop()] = arr[i];
    }
    stack.push(i);
}
```

**Common Bugs:** storing values instead of indices (loses position info needed for distance-based answers); wrong comparison direction (increasing vs decreasing stack).

**Edge Cases:** strictly decreasing array (no next greater for any), all equal elements, single element.

**Variations:** circular array (iterate `2n` times with modulo), largest rectangle in histogram (stack of indices with heights), trapping rain water (stack-based alternative to two-pointer).

**Related:** [[07-Arrays-Stack-Queue-Heap#Monotonic Queue|Monotonic Queue]]

**Must Solve:**
- Easy: —
- Medium: Daily Temperatures (739), Next Greater Element I/II (496, 503)
- Hard: Largest Rectangle in Histogram (84), Trapping Rain Water (42)

---

## Monotonic Queue (Deque-based)

> [!tip] When to use
> "Sliding window maximum/minimum" in O(n) — the deque-of-indices trick.

**Intuition:** Maintain a deque of indices with strictly decreasing values (for max); front is always the current window's max. Pop from back while smaller than incoming; pop from front when out of window range.

**Visual:**
```
[1,3,-1,-3,5,3,6,7], k=3
i=0(1): deque=[0]
i=1(3): pop 0 (1<3), deque=[1]
i=2(-1): deque=[1,2]  window=[0,2] max=arr[1]=3
i=3(-3): deque=[1,2,3] window=[1,3] max=arr[1]=3
i=4(5): pop 3,2,1 (all <5), deque=[4] window=[2,4] max=5
...
```

**Complexity:** Time O(n) · Space O(k)

**Template:**
```java
Deque<Integer> deque = new ArrayDeque<>(); // stores indices, values decreasing
int[] result = new int[n - k + 1];
for (int i = 0; i < n; i++) {
    if (!deque.isEmpty() && deque.peekFirst() <= i - k) deque.pollFirst(); // out of window
    while (!deque.isEmpty() && arr[deque.peekLast()] < arr[i]) deque.pollLast();
    deque.offerLast(i);
    if (i >= k - 1) result[i - k + 1] = arr[deque.peekFirst()];
}
```

**Common Bugs:** checking window-expiry condition with wrong bound (`<=` vs `<`); forgetting the result only starts once `i >= k-1`.

**Edge Cases:** k = 1 (every element is its own max), k = n (single window).

**Related:** [[04-Arrays-Windows-Search#Sliding Window Fixed|Sliding Window Fixed]], [[07-Arrays-Stack-Queue-Heap#Monotonic Stack|Monotonic Stack]]

**Must Solve:** Hard: Sliding Window Maximum (239)

---

## Deque (General)

> [!tip] When to use
> Need O(1) insertion/removal from **both ends** — palindrome checks, BFS with 0/1 weights, implementing the monotonic queue above.

**Template:**
```java
Deque<Integer> dq = new ArrayDeque<>();
dq.offerFirst(x); dq.offerLast(x);
dq.pollFirst(); dq.pollLast();
dq.peekFirst(); dq.peekLast();
```

**Complexity:** All operations O(1) amortized.

**Common Bugs:** using `LinkedList` when `ArrayDeque` is faster (no `null` allowed in `ArrayDeque`, and it's faster generally); confusing `push`/`pop` (stack semantics, operate on the **first**) with `offer`/`poll` (queue semantics, operate on **both ends** explicitly).

**Related:** [[07-Arrays-Stack-Queue-Heap#Monotonic Queue|Monotonic Queue]]

**Must Solve:** Medium: 0-1 Matrix (BFS with deque, 542) · Hard: Sliding Window Maximum (239)

---

## Heap (Priority Queue)

> [!tip] When to use
> "Kth largest/smallest", "top K frequent", "merge K sorted lists", "median of stream".

**Intuition:** A binary tree stored as an array, maintaining heap-order property (parent ≤/≥ children). Insert/extract-min(max) in O(log n).

**Visual:**
```
Min-Heap array: [1,3,2,7,5,4]
        1
      /   \
     3     2
    / \   /
   7   5 4
```

**Complexity:** Insert O(log n) · Peek O(1) · Extract O(log n) · Build from array O(n)

**Template:**
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // natural order (min at top)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
PriorityQueue<int[]> customHeap = new PriorityQueue<>((a, b) -> a[0] - b[0]); // by first element

// Top K frequent pattern
PriorityQueue<Map.Entry<Integer,Integer>> pq =
    new PriorityQueue<>((a, b) -> a.getValue() - b.getValue()); // min-heap by frequency
for (var e : freqMap.entrySet()) {
    pq.offer(e);
    if (pq.size() > k) pq.poll();  // keep only top k
}
```

**Common Bugs:** forgetting Java's `PriorityQueue` is a **min-heap by default** (need `Collections.reverseOrder()` or negate comparator for max-heap); assuming iteration order matches heap order (it doesn't — only `poll()` gives sorted order).

**Edge Cases:** k larger than array size, all equal elements, empty input.

**Variations:** two-heap technique for median of stream (max-heap for lower half, min-heap for upper half, rebalance on insert).

**Related:** [[05-Arrays-Sorting-Selection#Quick Select|Quick Select]] (alternative for Kth element, O(n) avg vs O(n log k))

**Must Solve:**
- Easy: Kth Largest Element in a Stream (703)
- Medium: Top K Frequent Elements (347), K Closest Points to Origin (973)
- Hard: Find Median from Data Stream (295), Merge k Sorted Lists (23)

---

## Bucket Sort

> [!tip] When to use
> Values fall into a **known bounded range**; need linear-time sort or frequency-based grouping (top K frequent).

**Template:**
```java
// Top K Frequent using bucket sort (bucket index = frequency)
List<Integer>[] buckets = new List[n + 1];
for (var e : freqMap.entrySet()) {
    int freq = e.getValue();
    if (buckets[freq] == null) buckets[freq] = new ArrayList<>();
    buckets[freq].add(e.getKey());
}
List<Integer> result = new ArrayList<>();
for (int i = n; i >= 0 && result.size() < k; i--) {
    if (buckets[i] != null) result.addAll(buckets[i]);
}
```

**Complexity:** Time O(n) · Space O(n)

**Common Bugs:** bucket array sized incorrectly (frequency can be at most `n`, so size `n+1`).

**Related:** [[07-Arrays-Stack-Queue-Heap#Heap|Heap]] (alternative for top-K, O(n log k))

**Must Solve:** Medium: Top K Frequent Elements (347), Sort Characters By Frequency (451)

---

## Counting Sort

> [!tip] When to use
> Small, known integer range — sort in O(n + range) without comparisons.

**Template:**
```java
int[] count = new int[maxVal + 1];
for (int x : arr) count[x]++;
int idx = 0;
for (int v = 0; v <= maxVal; v++) {
    while (count[v]-- > 0) arr[idx++] = v;
}
```

**Complexity:** Time O(n + k) where k = value range · Space O(k)

**Common Bugs:** using this on a huge/unbounded range (wastes memory) — only viable when range is small relative to n.

**Related:** [[04-Arrays-Windows-Search#Dutch National Flag|Dutch National Flag]] (special case, k=3)

**Must Solve:** Medium: Sort Colors (75)

---

## Merge Sort

> [!tip] When to use
> Need **guaranteed** O(n log n), **stability**, or as a building block (count inversions, external sorting, merge k sorted lists).

**Intuition:** Divide the array in half recursively, sort each half, merge the two sorted halves.

**Complexity:** Time O(n log n) always · Space O(n) · **Stable**

**Template:**
```java
void mergeSort(int[] arr, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, lo, mid);
    mergeSort(arr, mid + 1, hi);
    merge(arr, lo, mid, hi);
}
void merge(int[] arr, int lo, int mid, int hi) {
    int[] temp = new int[hi - lo + 1];
    int i = lo, j = mid + 1, k = 0;
    while (i <= mid && j <= hi) temp[k++] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= hi) temp[k++] = arr[j++];
    System.arraycopy(temp, 0, arr, lo, temp.length);
}
```

**Common Bugs:** `arr[i] <= arr[j]` vs `arr[i] < arr[j]` (matters for stability — use `<=` to keep left-side element first on ties).

**Related:** counting inversions uses the merge step to count cross-half inversions in O(n log n).

**Must Solve:** Hard: Count of Smaller Numbers After Self (315), Reverse Pairs (493)

---

## Quick Sort

> [!tip] When to use
> Need fast **average-case** in-place sort; not needed when stability or worst-case guarantees matter (use Merge Sort instead).

**Complexity:** Time O(n log n) average, O(n²) worst · Space O(log n) (recursion) · **Not stable**

**Template:**
```java
void quickSort(int[] arr, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition(arr, lo, hi);
    quickSort(arr, lo, p - 1);
    quickSort(arr, p + 1, hi);
}
// see partition() in Quick Select above
```

**Common Bugs:** deterministic pivot choice (e.g., always last element) causes O(n²) on sorted/reverse-sorted input — randomize the pivot.

**Related:** [[05-Arrays-Sorting-Selection#Quick Select|Quick Select]] (uses the same partition logic)

**Must Solve:** Medium: Sort an Array (912)

Next: [[08-Arrays-Advanced-DS]]
