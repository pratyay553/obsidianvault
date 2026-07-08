---
tags: [dsa, strings, kmp, rabin-karp, trie, z-algorithm]
---

# 11. Strings — Advanced

⬅ [[00-Home]] | Prev: [[10-Strings-Foundations]] | Next: [[12-Comparison-Tables]]

---

## KMP (Knuth-Morris-Pratt)

> [!tip] When to use
> Exact pattern matching in O(n+m), guaranteed — no hash collisions to worry about (unlike Rabin-Karp).

**Intuition:** Precompute a "failure function" (`lps` — longest proper prefix that's also a suffix) for the pattern, so on a mismatch you skip re-comparing characters you already know match.

**Visual:**
```
pattern: "ABABC"
lps:      [0,0,1,2,0]
  "A"->0, "AB"->0, "ABA"->1 (A matches), "ABAB"->2 (AB matches), "ABABC"->0
```

**Complexity:** Build lps O(m) · Search O(n) · Total O(n+m) · Space O(m)

**Template:**
```java
int[] buildLPS(String pattern) {
    int[] lps = new int[pattern.length()];
    int len = 0, i = 1;
    while (i < pattern.length()) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            lps[i++] = ++len;
        } else if (len != 0) {
            len = lps[len - 1];
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}

int kmpSearch(String text, String pattern) {
    int[] lps = buildLPS(pattern);
    int i = 0, j = 0;
    while (i < text.length()) {
        if (text.charAt(i) == pattern.charAt(j)) { i++; j++; }
        if (j == pattern.length()) return i - j;       // match found
        else if (i < text.length() && text.charAt(i) != pattern.charAt(j)) {
            if (j != 0) j = lps[j - 1];
            else i++;
        }
    }
    return -1;
}
```

**Common Bugs:** off-by-one in the lps table build; forgetting to fall back `j = lps[j-1]` on mismatch (causes O(n·m) degeneration, defeating the purpose).

**Edge Cases:** pattern longer than text, empty pattern, pattern with all identical characters (e.g., "aaaa").

**Related:** [[11-Strings-Advanced#Z-Algorithm|Z-Algorithm]] (alternative O(n+m) approach), [[11-Strings-Advanced#Rabin-Karp|Rabin-Karp]]

**Must Solve:** Medium: Implement strStr() / Find the Index of the First Occurrence (28) · Hard: Shortest Palindrome (214, uses KMP's lps trick)

---

## Rabin-Karp

> [!tip] When to use
> Multiple pattern search, plagiarism/duplicate substring detection — uses **rolling hash** to compare in O(1) per window after O(m) setup.

**Intuition:** Compute a hash of the pattern and of each window of the text; only do a full character comparison when hashes match (to rule out collisions).

**Complexity:** Average O(n+m) · Worst case O(n·m) (many hash collisions) · Space O(1)

**Template:**
```java
long BASE = 256, MOD = 1_000_000_007L;
boolean rabinKarp(String text, String pattern) {
    int n = text.length(), m = pattern.length();
    if (m > n) return false;
    long patternHash = 0, windowHash = 0, h = 1;
    for (int i = 0; i < m - 1; i++) h = (h * BASE) % MOD;
    for (int i = 0; i < m; i++) {
        patternHash = (patternHash * BASE + pattern.charAt(i)) % MOD;
        windowHash = (windowHash * BASE + text.charAt(i)) % MOD;
    }
    for (int i = 0; i <= n - m; i++) {
        if (patternHash == windowHash && text.substring(i, i + m).equals(pattern)) return true;
        if (i < n - m) {
            windowHash = (BASE * (windowHash - text.charAt(i) * h) + text.charAt(i + m)) % MOD;
            if (windowHash < 0) windowHash += MOD;
        }
    }
    return false;
}
```

**Common Bugs:** forgetting the modulo can go negative in Java (must add `MOD` back); not verifying with a full string comparison after a hash match (collision risk); using a weak modulus (small prime → collisions).

**Related:** [[11-Strings-Advanced#Rolling Hash|Rolling Hash]] (the technique Rabin-Karp is built on)

**Must Solve:** Medium: Repeated DNA Sequences (187) · Hard: Distinct Echo Substrings

---

## Z-Algorithm

> [!tip] When to use
> Pattern matching, finding all occurrences, computing border/period information — alternative to KMP, often considered more intuitive.

**Intuition:** `Z[i]` = length of the longest substring starting at `i` that matches a prefix of the string. Build `pattern + '$' + text` and scan for positions where `Z[i] == pattern.length()`.

**Complexity:** Time O(n+m) · Space O(n+m)

**Template:**
```java
int[] zFunction(String s) {
    int n = s.length();
    int[] z = new int[n];
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = Math.min(r - i, z[i - l]);
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
// Usage: combined = pattern + "$" + text; find indices where z[i] == pattern.length()
```

**Common Bugs:** forgetting the separator `$` (must not appear in either string) — without it, matches can bleed across pattern/text boundary.

**Related:** [[11-Strings-Advanced#KMP (Knuth-Morris-Pratt)|KMP]]

**Must Solve:** Medium: Implement strStr() (28, Z-algo variant) · Hard: Shortest Palindrome (214)

---

## Rolling Hash

> [!tip] When to use
> Building block for Rabin-Karp; also useful for "longest duplicate substring", "distinct substrings count" via binary search + hashing.

**Intuition:** Represent a string as a polynomial hash so that sliding the window by one character updates the hash in O(1) (remove leading term's contribution, shift, add new term).

**Template:**
```java
long hash = 0, power = 1;
long BASE = 31, MOD = 1_000_000_007L;
// building initial hash of length-m window, then for each shift:
hash = (hash - (s.charAt(i) - 'a' + 1) * power % MOD + MOD) % MOD; // remove leftmost
hash = (hash * BASE + (s.charAt(i + m) - 'a' + 1)) % MOD;           // add new rightmost
```

**Common Bugs:** single modulus increases collision risk on adversarial inputs — for contest-safe code, use **double hashing** (two different bases/mods) or a randomized base.

**Related:** [[11-Strings-Advanced#Rabin-Karp|Rabin-Karp]]

**Must Solve:** Hard: Longest Duplicate Substring (1044)

---

## Trie (Prefix Tree) Basics

> [!tip] When to use
> "Autocomplete", "word search", "longest common prefix among many words", dictionary/prefix membership queries.

**Intuition:** A tree where each path from root spells a prefix; each node holds children (one per possible next character) and an `isEndOfWord` flag.

**Visual:**
```
Insert "cat", "car", "dog":
root
 ├─ c
 │   └─ a
 │       ├─ t (end)
 │       └─ r (end)
 └─ d
     └─ o
         └─ g (end)
```

**Complexity:** Insert/Search O(L) where L = word length · Space O(alphabet_size × total_characters)

**Template:**
```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd = false;
}
class Trie {
    TrieNode root = new TrieNode();
    void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }
    boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }
    boolean startsWith(String prefix) { return find(prefix) != null; }
    private TrieNode find(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

**Common Bugs:** forgetting `isEnd` flag (can't distinguish a full word from a prefix); array size mismatch for non-lowercase input.

**Related:** [[06-Arrays-Hashing-Greedy#HashMap|HashMap]] (simpler alternative when prefix-sharing isn't needed)

**Must Solve:** Medium: Implement Trie (208), Word Search II (212) · Hard: —

---

## Suffix Array (Overview)

> [!tip] When to use
> Multiple substring queries, longest repeated substring, distinct-substring counting — advanced/competitive level.

**Intuition:** Sort all suffixes of a string lexicographically; store their starting indices. Binary search over this array answers many substring queries efficiently.

**Complexity:** Naive build O(n² log n) · Optimized (e.g., SA-IS, doubling technique) O(n log n)

> [!note] Interview reality
> Rarely hand-coded live; know the concept and that it pairs with the **LCP array** for O(n log n) longest-repeated-substring solutions.

**Related:** [[11-Strings-Advanced#LCP Array (Overview)|LCP Array]]

**Must Solve:** Hard: Longest Duplicate Substring (1044, suffix array approach)

---

## LCP Array (Overview)

> [!tip] When to use
> Paired with a suffix array — `LCP[i]` = length of the longest common prefix between adjacent suffixes in sorted order. Used for longest repeated substring, counting distinct substrings.

**Complexity:** Built in O(n) from a suffix array via Kasai's algorithm (given the suffix array is already built).

**Related:** [[11-Strings-Advanced#Suffix Array (Overview)|Suffix Array]]

---

## Lexicographical Algorithms

> [!tip] When to use
> "Smallest/largest string after operation", "next permutation", "remove k digits to form smallest number."

**Template (Next Permutation):**
```java
void nextPermutation(int[] nums) {
    int i = nums.length - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;
    if (i >= 0) {
        int j = nums.length - 1;
        while (nums[j] <= nums[i]) j--;
        swap(nums, i, j);
    }
    reverse(nums, i + 1, nums.length - 1);
}
```

**Template (Remove K Digits — monotonic stack for smallest number):**
```java
String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : num.toCharArray()) {
        while (!stack.isEmpty() && k > 0 && stack.peek() > c) { stack.pop(); k--; }
        stack.push(c);
    }
    while (k-- > 0) stack.pop();
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) sb.append(stack.pop());
    sb.reverse();
    while (sb.length() > 1 && sb.charAt(0) == '0') sb.deleteCharAt(0);
    return sb.length() == 0 ? "0" : sb.toString();
}
```

**Related:** [[07-Arrays-Stack-Queue-Heap#Monotonic Stack|Monotonic Stack]]

**Must Solve:** Medium: Next Permutation (31), Remove K Digits (402)

---

## StringBuilder Tricks

> [!tip] When to use
> Any repeated string concatenation — `String + String` in a loop is O(n²); `StringBuilder` is O(n).

```java
StringBuilder sb = new StringBuilder();
sb.append(x);                       // O(1) amortized
sb.reverse();                       // O(n), useful for building answers backwards
sb.deleteCharAt(sb.length() - 1);   // O(1) remove last (backtracking use case)
sb.setCharAt(i, c);                  // O(1) in-place mutation
```

> [!warning] Common Interview Mistake
> Doing `result = result + s.charAt(i)` inside a loop → silently O(n²). Always use `StringBuilder` for loop-based string building.

**Must Solve:** Any string-building/backtracking problem (e.g., Generate Parentheses, 22)

---

## ASCII Tricks

> [!tip] Quick reference
```java
c - '0'          // char digit -> int value
c - 'a'          // lowercase letter -> 0-25 index
c - 'A'          // uppercase letter -> 0-25 index
(char)('a' + i)  // index -> lowercase letter
Character.toLowerCase(c) / toUpperCase(c)
Character.isDigit(c) / isLetter(c) / isLetterOrDigit(c) / isWhitespace(c)
c ^ 32           // toggles case bit for ASCII letters (A<->a), fast but less readable
```

**Common Bugs:** using ASCII tricks on non-ASCII/Unicode input — always confirm constraints first.

---

## Unicode Notes

> [!warning] Know this before you assume `char == codepoint`
> - Java `char` is a **UTF-16 code unit**, not a Unicode code point. Characters outside the Basic Multilingual Plane (emoji, some CJK) are **surrogate pairs** — 2 `char`s per 1 visual character.
> - Use `String.codePointAt`, `str.codePoints()`, or iterate with `int cp = str.codePointAt(i); i += Character.charCount(cp);` when Unicode-correctness matters.
> - `s.length()` counts UTF-16 code units, **not** visible characters, when surrogate pairs are present.

**Must Solve:** Rare in typical interviews unless explicitly stated ("handle emoji/Unicode").

---

## Regex (Interview Level)

> [!tip] When it's acceptable to reach for regex
> Simple validation (numeric string, email-shape check) or quick splitting — avoid for complex parsing logic where interviewers want to see your algorithmic thinking, not a one-liner.

```java
s.matches("[a-zA-Z]+");          // full-string match
s.split("\\s+");                  // split on whitespace
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher(s);
while (m.find()) { /* m.group() */ }
```

**Common Bugs:** forgetting `matches()` requires matching the **entire** string (use `find()` for partial matches); catastrophic backtracking on pathological patterns (rare in interviews but know it exists).

**Must Solve:** Medium: Valid Number (65), Regular Expression Matching (10 — usually solved with DP, not `java.util.regex`)

Next: [[12-Comparison-Tables]]
