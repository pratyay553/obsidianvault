---
tags: [dsa, strings, palindrome, sliding-window]
---

# 10. Strings — Foundations

⬅ [[00-Home]] | Prev: [[09-Matrix-Grid]] | Next: [[11-Strings-Advanced]]

---

## Character Frequency

> [!tip] When to use
> Anagram checks, "most/least frequent character", character-count-based comparisons.

**Template:**
```java
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;
```

**Complexity:** Time O(n) · Space O(1) (fixed alphabet size)

**Common Bugs:** wrong offset for uppercase/mixed case; assuming ASCII when input is Unicode.

**Related:** [[06-Arrays-Hashing-Greedy#Frequency Array|Frequency Array]]

**Must Solve:** Easy: Valid Anagram (242), First Unique Character (387)

---

## Sliding Window (Strings)

> [!tip] When to use
> "Longest/shortest substring with property X" — same mechanics as array sliding window, applied to characters.

**Template (Minimum Window Substring skeleton):**
```java
Map<Character, Integer> need = new HashMap<>(), window = new HashMap<>();
for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);
int left = 0, valid = 0, start = 0, minLen = Integer.MAX_VALUE;
for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    if (need.containsKey(c)) {
        window.merge(c, 1, Integer::sum);
        if (window.get(c).intValue() == need.get(c).intValue()) valid++;
    }
    while (valid == need.size()) {
        if (right - left + 1 < minLen) { minLen = right - left + 1; start = left; }
        char d = s.charAt(left);
        if (need.containsKey(d)) {
            if (window.get(d).intValue() == need.get(d).intValue()) valid--;
            window.put(d, window.get(d) - 1);
        }
        left++;
    }
}
return minLen == Integer.MAX_VALUE ? "" : s.substring(start, start + minLen);
```

**Common Bugs:** comparing map sizes instead of matched-character counts (`valid == need.size()`); forgetting `window.get(d) - 1` can go negative if not careful with extra chars.

**Related:** [[04-Arrays-Windows-Search#Sliding Window Variable|Sliding Window Variable]] (identical pattern, array version)

**Must Solve:** Hard: Minimum Window Substring (76) · Medium: Longest Substring Without Repeating Characters (3), Permutation in String (567)

---

## Anagram

> [!tip] When to use
> "Are two strings anagrams", "group anagrams", "find all anagram substrings."

**Intuition:** Two strings are anagrams iff they have identical character frequency counts. Use a sorted string or a frequency-count tuple as the canonical key.

**Template:**
```java
boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] freq = new int[26];
    for (char c : a.toCharArray()) freq[c - 'a']++;
    for (char c : b.toCharArray()) if (--freq[c - 'a'] < 0) return false;
    return true;
}

// Group Anagrams: use sorted string as map key
Map<String, List<String>> groups = new HashMap<>();
for (String w : words) {
    char[] chars = w.toCharArray();
    Arrays.sort(chars);
    groups.computeIfAbsent(new String(chars), k -> new ArrayList<>()).add(w);
}
```

**Complexity:** Per-check O(n) (frequency method) or O(n log n) (sort method) · Space O(1)/O(n)

**Common Bugs:** forgetting the length check first (saves work); using sort-based key when frequency-count key would be faster for long strings.

**Related:** [[10-Strings-Foundations#Sliding Window (Strings)|Sliding Window]] (for "find all anagram substrings in a string")

**Must Solve:** Easy: Valid Anagram (242) · Medium: Group Anagrams (49), Find All Anagrams in a String (438)

---

## Palindrome

> [!tip] When to use
> "Is this a palindrome", validate ignoring case/non-alphanumeric.

**Template:**
```java
boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l) != s.charAt(r)) return false;
        l++; r--;
    }
    return true;
}
```

**Complexity:** Time O(n) · Space O(1)

**Common Bugs:** forgetting to filter non-alphanumeric / normalize case first (LC 125-style variant); off-by-one on odd vs even length.

**Related:** [[10-Strings-Foundations#Expand Around Center|Expand Around Center]], [[03-Arrays-Foundations#Two Pointer|Two Pointer]]

**Must Solve:** Easy: Valid Palindrome (125) · Medium: Palindromic Substrings (647) · Hard: —

---

## Expand Around Center

> [!tip] When to use
> "Longest palindromic substring" — O(n²) approach, simpler than Manacher's.

**Intuition:** Every palindrome has a center (a single character for odd length, between two characters for even length). Expand outward from each of the `2n-1` possible centers.

**Visual:**
```
"babad" - center at index 1 ('a'):
expand: b[a]b -> match -> b[a]b -> "bab" (expand once more fails, 'b' vs boundary)
Try center at index 2 ('b'): a[b]a -> "aba"
Longest found: "bab" or "aba" (both length 3)
```

**Complexity:** Time O(n²) · Space O(1)

**Template:**
```java
String longestPalindrome(String s) {
    int start = 0, maxLen = 0;
    for (int center = 0; center < s.length(); center++) {
        int len1 = expand(s, center, center);       // odd length
        int len2 = expand(s, center, center + 1);    // even length
        int len = Math.max(len1, len2);
        if (len > maxLen) {
            maxLen = len;
            start = center - (len - 1) / 2;
        }
    }
    return s.substring(start, start + maxLen);
}
int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
```

**Common Bugs:** forgetting to check both odd-length and even-length centers; off-by-one computing `start` from `center` and `len`.

**Related:** [[10-Strings-Foundations#Manacher|Manacher]] (O(n) upgrade)

**Must Solve:** Medium: Longest Palindromic Substring (5), Palindromic Substrings (647)

---

## Manacher's Algorithm

> [!tip] When to use
> "Longest palindromic substring" but need **O(n)**, not O(n²) — rare in typical interviews, common in advanced/competitive rounds.

**Intuition:** Insert separators between characters to unify odd/even cases, then use previously computed palindrome radii + mirror symmetry to avoid re-expanding from scratch at every center.

**Complexity:** Time O(n) · Space O(n)

**Template (conceptual skeleton):**
```java
// Transform: "aba" -> "^#a#b#a#$" (sentinels avoid bounds checks)
// P[i] = radius of palindrome centered at i in the transformed string
// Use center C and right boundary R of the rightmost known palindrome:
//   if i < R: P[i] = min(R - i, P[mirror of i])  (mirror = 2*C - i)
//   then attempt to expand further from there
// Track the center with max P[i] -> map back to original string indices
```

> [!warning] Interview reality check
> Most interviewers accept **Expand Around Center O(n²)** for this problem. Only bring up Manacher if asked explicitly for O(n), and be ready to explain (not necessarily hand-code perfectly) the mirror trick.

**Related:** [[10-Strings-Foundations#Expand Around Center|Expand Around Center]]

**Must Solve:** Medium/Hard: Longest Palindromic Substring (5) — O(n) variant

Next: [[11-Strings-Advanced]]
