---
tags: [dsa, arrays, hashing, greedy]
---

# 6. Arrays — Hashing & Greedy

⬅ [[00-Home]] | Prev: [[05-Arrays-Sorting-Selection]] | Next: [[07-Arrays-Stack-Queue-Heap]]

---

## HashMap

> [!tip] When to use
> Need O(1) average lookup by key: counting, complement search (Two Sum), grouping, caching.

**Intuition:** Hash the key to a bucket; store key→value pairs; collisions handled via chaining/open addressing internally.

**Template:**
```java
Map<Integer, Integer> map = new HashMap<>();
for (int i = 0; i < arr.length; i++) {
    int complement = target - arr[i];
    if (map.containsKey(complement)) return new int[]{map.get(complement), i};
    map.put(arr[i], i);
}
```

**Complexity:** Average O(1) get/put · Worst case O(n) (hash collisions) · Space O(n)

**Common Bugs:** using the same element twice (Two Sum — check complement *before* inserting current); assuming iteration order (use `LinkedHashMap` if order matters).

**Edge Cases:** empty map, duplicate keys (last write wins on `put`), null keys (allowed once in `HashMap`, not in `Hashtable`).

**Related:** [[06-Arrays-Hashing-Greedy#HashSet|HashSet]]

**Must Solve:** Easy: Two Sum (1) · Medium: Group Anagrams (49), Subarray Sum Equals K (560) · Hard: —

---

## HashSet

> [!tip] When to use
> Need O(1) membership check, deduplication, "contains" queries without associated values.

**Template:**
```java
Set<Integer> seen = new HashSet<>();
for (int x : arr) {
    if (!seen.add(x)) { /* duplicate found */ }
}
```

**Complexity:** Average O(1) add/contains · Space O(n)

**Common Bugs:** forgetting `Set.add()` returns `false` if element already present (useful for one-line duplicate detection); using a `List.contains()` by mistake (O(n) instead of O(1)).

**Edge Cases:** empty set, all duplicates, all unique.

**Related:** [[06-Arrays-Hashing-Greedy#HashMap|HashMap]], [[05-Arrays-Sorting-Selection#Cyclic Sort|Cyclic Sort]] (O(1) space alternative for range-limited values)

**Must Solve:** Easy: Contains Duplicate (217) · Medium: Longest Consecutive Sequence (128) · Hard: —

---

## Frequency Array

> [!tip] When to use
> Known, small alphabet/range (e.g., lowercase letters, digits 0-9) — faster and lighter than a HashMap.

**Template:**
```java
int[] freq = new int[26]; // for lowercase a-z
for (char c : s.toCharArray()) freq[c - 'a']++;
```

**Complexity:** Time O(n) · Space O(1) (fixed-size array, e.g. 26 or 256)

**Advantages:** Faster constant factor than HashMap; no boxing/hashing overhead.
**Limitations:** Only works for small, known, contiguous ranges.

**Common Bugs:** wrong base character (`c - 'a'` vs `c - 'A'`); array too small for Unicode/mixed-case input.

**Edge Cases:** empty string, string with characters outside expected range (validate first!), case sensitivity.

**Related:** [[10-Strings-Foundations#Character Frequency|Character Frequency]]

**Must Solve:** Easy: Valid Anagram (242), Ransom Note (383) · Medium: Group Anagrams (49) · Hard: —

---

## Greedy

> [!tip] When to use
> Locally optimal choice at each step leads to a globally optimal solution — usually provable via exchange argument or matroid structure. Common in scheduling, intervals, jump/reachability problems.

**Intuition:** Sort or scan with a rule ("always pick the earliest deadline", "always extend reach as far as possible") and never backtrack.

**Visual (Jump Game):**
```
nums = [2,3,1,1,4]
maxReach=0
i=0: maxReach=max(0,0+2)=2
i=1: maxReach=max(2,1+3)=4  (>= last index 4) -> can reach!
```

**Complexity:** Usually O(n) or O(n log n) (if sorting needed) · Space O(1)

**Template (reachability greedy):**
```java
int maxReach = 0;
for (int i = 0; i < nums.length; i++) {
    if (i > maxReach) return false;  // stuck, can't reach index i
    maxReach = Math.max(maxReach, i + nums[i]);
}
return true;
```

**Common Bugs:** assuming greedy works without proving the exchange argument (test on adversarial cases); forgetting to sort by the correct key first (e.g., end time, not start time, for activity selection).

**Edge Cases:** single element, all zeros (can't move), first element is 0.

**Related:** [[05-Arrays-Sorting-Selection#Merge Intervals|Merge Intervals]] (activity selection greedy)

**Must Solve:**
- Easy: Best Time to Buy/Sell Stock (121), Assign Cookies (455)
- Medium: Jump Game (55), Gas Station (134), Candy (135)
- Hard: Jump Game II (45 — greedy O(n) solution), Candy (135, tightened proof)

Next: [[07-Arrays-Stack-Queue-Heap]]
