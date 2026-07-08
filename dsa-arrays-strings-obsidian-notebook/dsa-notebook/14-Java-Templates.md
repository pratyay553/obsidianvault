---
tags: [dsa, java, templates]
---

# 14. Java Templates — All in One Place

⬅ [[00-Home]] | Prev: [[13-Complexity-Cheat-Sheet]] | Next: [[15-Common-Tricks]]

> [!tip] Purpose
> Copy-paste starting points during a live interview. Each is intentionally minimal — adapt variable names to the problem.

## Fast Input (competitive-style)

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
StreamTokenizer in = new StreamTokenizer(br);
in.nextToken(); int n = (int) in.nval;

// Or simple Scanner (slower, fine for interviews):
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
```

## Prefix Sum

```java
int[] prefix = new int[n + 1];
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + arr[i];
int rangeSum = prefix[r + 1] - prefix[l];
```

## Sliding Window (Variable)

```java
int left = 0, best = 0;
for (int right = 0; right < n; right++) {
    // add arr[right] to window state
    while (/* window invalid */ false) {
        // remove arr[left] from window state
        left++;
    }
    best = Math.max(best, right - left + 1);
}
```

## Binary Search (classic / lower bound / upper bound)

```java
// classic
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}

// lower bound: first index with arr[i] >= target
int lo2 = 0, hi2 = n;
while (lo2 < hi2) {
    int mid = lo2 + (hi2 - lo2) / 2;
    if (arr[mid] < target) lo2 = mid + 1; else hi2 = mid;
}

// upper bound: first index with arr[i] > target
int lo3 = 0, hi3 = n;
while (lo3 < hi3) {
    int mid = lo3 + (hi3 - lo3) / 2;
    if (arr[mid] <= target) lo3 = mid + 1; else hi3 = mid;
}
```

## Kadane

```java
int maxSoFar = arr[0], maxEndingHere = arr[0];
for (int i = 1; i < n; i++) {
    maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
    maxSoFar = Math.max(maxSoFar, maxEndingHere);
}
```

## Merge Intervals

```java
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
List<int[]> merged = new ArrayList<>();
for (int[] cur : intervals) {
    if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < cur[0]) merged.add(cur);
    else merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], cur[1]);
}
```

## Monotonic Stack (next greater element)

```java
int[] result = new int[n];
Arrays.fill(result, -1);
Deque<Integer> stack = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) result[stack.pop()] = arr[i];
    stack.push(i);
}
```

## Heap (min/max/custom)

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
PriorityQueue<int[]> byFirst = new PriorityQueue<>((a, b) -> a[0] - b[0]);
```

## Frequency Map

```java
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
```

## Trie

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}
```

## KMP (lps table)

```java
int[] buildLPS(String p) {
    int[] lps = new int[p.length()];
    int len = 0, i = 1;
    while (i < p.length()) {
        if (p.charAt(i) == p.charAt(len)) lps[i++] = ++len;
        else if (len != 0) len = lps[len - 1];
        else lps[i++] = 0;
    }
    return lps;
}
```

## Rabin-Karp (rolling hash core)

```java
long BASE = 256, MOD = 1_000_000_007L;
hash = (hash * BASE + s.charAt(i)) % MOD;
```

## Z-Algorithm

```java
int[] zFunction(String s) {
    int n = s.length(), l = 0, r = 0;
    int[] z = new int[n];
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = Math.min(r - i, z[i - l]);
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

## Quick Select

```java
int quickSelect(int[] arr, int lo, int hi, int k) {
    int p = partition(arr, lo, hi);
    if (p == k) return arr[p];
    return p < k ? quickSelect(arr, p + 1, hi, k) : quickSelect(arr, lo, p - 1, k);
}
```

## Matrix (4-directional traversal)

```java
int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
for (int[] d : dirs) {
    int nr = r + d[0], nc = c + d[1];
    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) { /* visit */ }
}
```

## Union Find

```java
int[] parent;
int find(int x) { return parent[x] == x ? x : (parent[x] = find(parent[x])); }
void union(int a, int b) { parent[find(a)] = find(b); }
```

## Deque (as monotonic queue)

```java
Deque<Integer> dq = new ArrayDeque<>();
while (!dq.isEmpty() && arr[dq.peekLast()] < arr[i]) dq.pollLast();
dq.offerLast(i);
if (dq.peekFirst() <= i - k) dq.pollFirst();
```

## Character Count (26-letter array)

```java
int[] count = new int[26];
for (char c : s.toCharArray()) count[c - 'a']++;
```

Next: [[15-Common-Tricks]]
