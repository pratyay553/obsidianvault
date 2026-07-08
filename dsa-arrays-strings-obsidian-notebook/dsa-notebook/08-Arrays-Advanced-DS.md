---
tags: [dsa, arrays, fenwick, segment-tree, union-find]
---

# 8. Arrays — Advanced Data Structures (Overview Level)

⬅ [[00-Home]] | Prev: [[07-Arrays-Stack-Queue-Heap]] | Next: [[09-Matrix-Grid]]

> [!note] Scope
> These are asked at senior/advanced interview levels or in competitive-programming-flavored rounds. Know **when to reach for them** and the **basic template** — deep implementation detail is rarely required live.

---

## Fenwick Tree (Binary Indexed Tree)

> [!tip] When to use
> Repeated **point updates + prefix sum queries** — better than plain Prefix Sum when the array changes frequently.

**Intuition:** Each index stores a partial sum covering a power-of-2-sized range; updates and queries walk the tree via `i & (-i)` bit tricks.

**Complexity:** Update O(log n) · Query O(log n) · Space O(n)

**Template:**
```java
int[] tree; int n;
void update(int i, int delta) {           // 1-indexed
    for (; i <= n; i += i & (-i)) tree[i] += delta;
}
int query(int i) {                          // prefix sum [1..i]
    int sum = 0;
    for (; i > 0; i -= i & (-i)) sum += tree[i];
    return sum;
}
int rangeSum(int l, int r) { return query(r) - query(l - 1); }
```

**Common Bugs:** using 0-indexed arrays directly (Fenwick trees are naturally 1-indexed); off-by-one in range queries.

**Related:** [[03-Arrays-Foundations#Prefix Sum|Prefix Sum]] (simpler, but O(n) to update), [[08-Arrays-Advanced-DS#Segment Tree (Overview)|Segment Tree]]

**Must Solve:** Hard: Range Sum Query - Mutable (307), Count of Smaller Numbers After Self (315)

---

## Segment Tree (Overview)

> [!tip] When to use
> Range queries (sum/min/max/gcd) **with updates**, or when you need more than sum (e.g., range min AND range update).

**Intuition:** A binary tree where each node covers a range; leaves are individual elements, internal nodes aggregate their children. Build O(n), query/update O(log n).

**Complexity:** Build O(n) · Query O(log n) · Update O(log n) · Space O(4n) typical array-based implementation

**Template (array-based, sum variant):**
```java
int[] tree; int n;
void build(int[] arr, int node, int lo, int hi) {
    if (lo == hi) { tree[node] = arr[lo]; return; }
    int mid = (lo + hi) / 2;
    build(arr, 2*node, lo, mid);
    build(arr, 2*node+1, mid+1, hi);
    tree[node] = tree[2*node] + tree[2*node+1];
}
int query(int node, int lo, int hi, int l, int r) {
    if (r < lo || hi < l) return 0;               // no overlap
    if (l <= lo && hi <= r) return tree[node];     // total overlap
    int mid = (lo + hi) / 2;
    return query(2*node, lo, mid, l, r) + query(2*node+1, mid+1, hi, l, r);
}
```

**Common Bugs:** array size too small (`4*n` is the safe convention); mixing up node indices when using 1-indexed tree array.

**Advantages over Fenwick Tree:** supports min/max/gcd, range updates with lazy propagation.
**Limitations:** more code, more memory than Fenwick for simple sum-only use cases.

**Related:** [[08-Arrays-Advanced-DS#Fenwick Tree (Binary Indexed Tree)|Fenwick Tree]], [[08-Arrays-Advanced-DS#Sparse Table (Overview)|Sparse Table]]

**Must Solve:** Hard: Range Sum Query - Mutable (307), Range Minimum Query variants

---

## Sparse Table (Overview)

> [!tip] When to use
> **Static** array (no updates), repeated range min/max/gcd queries — O(1) query after O(n log n) preprocessing.

**Intuition:** Precompute answers for all ranges of length `2^k` starting at every index; any range `[l,r]` is covered by two overlapping power-of-2 ranges (works for idempotent operations like min/max/gcd, not sum — overlap would double count).

**Complexity:** Build O(n log n) · Query O(1) · Space O(n log n)

**Template (Range Minimum Query):**
```java
int[][] sparse; int[] log;
void build(int[] arr) {
    int n = arr.length, LOG = (int)(Math.log(n) / Math.log(2)) + 1;
    sparse = new int[n][LOG];
    for (int i = 0; i < n; i++) sparse[i][0] = arr[i];
    for (int j = 1; (1 << j) <= n; j++)
        for (int i = 0; i + (1 << j) - 1 < n; i++)
            sparse[i][j] = Math.min(sparse[i][j-1], sparse[i + (1 << (j-1))][j-1]);
}
int query(int l, int r) {  // inclusive
    int j = (int)(Math.log(r - l + 1) / Math.log(2));
    return Math.min(sparse[l][j], sparse[r - (1 << j) + 1][j]);
}
```

**Common Bugs:** using this for range **sum** (doesn't work directly — overlapping ranges double-count; use prefix sum or Fenwick instead).

**Related:** [[08-Arrays-Advanced-DS#Segment Tree (Overview)|Segment Tree]] (supports updates, sparse table doesn't)

**Must Solve:** Hard: Range Minimum Query (conceptual, less common as standalone LeetCode problem)

---

## Union Find (Disjoint Set Union — Overview)

> [!tip] When to use
> "Connected components", "redundant connection", "accounts merge", dynamic connectivity queries.

**Intuition:** Each element points to a "parent"; two elements are connected if they share the same root. Union by rank/size + path compression gives near-O(1) amortized operations.

**Complexity:** Find/Union: O(α(n)) amortized (α = inverse Ackermann, effectively constant) · Space O(n)

**Template:**
```java
int[] parent, rank;
void init(int n) {
    parent = new int[n]; rank = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;
}
int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]); // path compression
    return parent[x];
}
void union(int a, int b) {
    int ra = find(a), rb = find(b);
    if (ra == rb) return;
    if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
    parent[rb] = ra;
    if (rank[ra] == rank[rb]) rank[ra]++;
}
```

**Common Bugs:** forgetting path compression (degrades to O(n) find in worst case); union by rank/size skipped (same issue).

**Related:** [[09-Matrix-Grid#BFS Grid|BFS/DFS Grid]] (alternative for connectivity in grids)

**Must Solve:** Medium: Number of Provinces (547), Redundant Connection (684), Accounts Merge (721) · Hard: —

Next: [[09-Matrix-Grid]]
