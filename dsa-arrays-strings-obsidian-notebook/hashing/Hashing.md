# Hashing in Java for DSA  🔥

Hashing is one of the **most important DSA concepts for coding interviews**. It helps you solve problems faster by using data structures such as `HashMap` and `HashSet`.

Let's understand hashing in Java with examples, use cases, and common interview questions.

## 1. What is hashing?

Hashing is a technique used to store and retrieve data efficiently.

For example, suppose you have this array:

```
int[] arr = {10, 20, 30, 10, 20, 10};
```

You want to count how many times each number occurs.

Without hashing, you might repeatedly scan the array, taking `O(n²)` time with a straightforward approach.

With a `HashMap`, you can count the frequencies in **O(n) average time**.

Expected result:

```
10 -> 3
20 -> 2
30 -> 1
```

### Why do we use hashing?

- To find elements quickly.
- To count frequencies.
- To detect duplicates.
- To solve Two Sum problems.
- To check whether an element exists.
- To group elements based on a common property.

---

## 2. Types of hashing data structures in Java

Java provides several classes that are useful for hashing.

|Data structure|Purpose|Example|
|---|---|---|
|`HashMap`|Store key-value pairs|Number → Frequency|
|`HashSet`|Store unique elements|Find duplicates|
|`LinkedHashMap`|Maintain insertion order|Ordered key-value data|
|`LinkedHashSet`|Maintain insertion order of unique elements|Remove duplicates while preserving order|
|`Hashtable`|Legacy synchronized key-value map|Older Java applications|

For DSA interviews, focus primarily on **`HashMap`** **and** **`HashSet`**.

## 3. HashMap in Java

A `HashMap` stores data in key-value pairs.

Syntax:

```
HashMap<KeyType, ValueType> map = new HashMap<>();
```

Example:

```
import java.util.HashMap;

public class Main {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();

        map.put("Apple", 10);
        map.put("Banana", 20);
        map.put("Mango", 30);

        System.out.println(map.get("Apple")); // 10
        System.out.println(map.get("Banana")); // 20

        System.out.println(map.containsKey("Mango")); // true
        System.out.println(map.containsKey("Orange")); // false
    }
}
```

### Important HashMap methods

|Method|Purpose|Example|
|---|---|---|
|`put(key, value)`|Add or update an entry|`map.put(10, 100)`|
|`get(key)`|Retrieve a value|`map.get(10)`|
|`containsKey(key)`|Check whether a key exists|`map.containsKey(10)`|
|`containsValue(value)`|Check whether a value exists|`map.containsValue(100)`|
|`getOrDefault(key, defaultValue)`|Return a default if the key is absent|`map.getOrDefault(10, 0)`|
|`remove(key)`|Remove an entry|`map.remove(10)`|
|`size()`|Count entries|`map.size()`|
|`isEmpty()`|Check whether the map is empty|`map.isEmpty()`|

**Interview tip:** `HashMap` allows one `null` key and multiple `null` values. It does not guarantee iteration order.

---

# 4. Use case 1: Count the frequency of array elements

**Interview problem:** Given an integer array, count the frequency of every element.

Input:

```
[1, 2, 3, 1, 2, 1]
```

Output:

```
1 -> 3
2 -> 2
3 -> 1
```

### Java solution

```
import java.util.HashMap;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 1, 2, 1};

        HashMap<Integer, Integer> freq = new HashMap<>();

        for (int num : arr) {
            freq.put(num, freq.getOrDefault(num, 0) + 1);
        }

        for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
            System.out.println(
                entry.getKey() + " -> " + entry.getValue()
            );
        }
    }
}
```

### How does it work?

Consider the expression:

```
freq.put(num, freq.getOrDefault(num, 0) + 1);
```

Suppose `num = 1`.

- First occurrence: `1 -> 1`
- Second occurrence: `1 -> 2`
- Third occurrence: `1 -> 3`

`getOrDefault(num, 0)` returns the existing frequency or `0` if the number hasn't appeared before.

**Time complexity:** `O(n)` average.

**Space complexity:** `O(k)`, where `k` is the number of distinct elements.

### When is this useful?

- Finding the most frequent element.
- Counting characters in a string.
- Finding duplicate numbers.
- Solving anagrams.
- Counting votes or occurrences.

---

# 5. Use case 2: Find duplicates using HashSet

**Interview problem:** Determine whether an array contains duplicate elements.

Input:

```
[1, 2, 3, 4, 2]
```

Output:

```
true
```

Because `2` occurs twice.

### Java solution

```
import java.util.HashSet;

public class Main {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 2};

        HashSet<Integer> set = new HashSet<>();

        for (int num : arr) {
            if (set.contains(num)) {
                System.out.println(true);
                return;
            }

            set.add(num);
        }

        System.out.println(false);
    }
}
```

### How does it work?

The set stores only unique elements.

1. Add `1`.
2. Add `2`.
3. Add `3`.
4. Add `4`.
5. Encounter `2` again.
6. Since `2` already exists, return `true`.

**Time complexity:** `O(n)` average.

**Space complexity:** `O(n)` in the worst case.

### A useful HashSet trick

You can also write:

```
if (!set.add(num)) {
    System.out.println("Duplicate found: " + num);
}
```

The `add()` method returns `false` when the element is already present.

**When to use it:** Whenever you only need to know whether an element exists and don't need to store an associated value.

---

# 6. Use case 3: Two Sum — a very common DSA interview problem

**Problem:** Given an array and a target, return the indices of two elements whose sum equals the target.

Input:

```
arr = [2, 7, 11, 15]
target = 9
```

Output:

```
[0, 1]
```

Because:

```
arr[0] + arr[1] = 2 + 7 = 9
```

### Approach 1: Brute force

```
for (int i = 0; i < arr.length; i++) {
    for (int j = i + 1; j < arr.length; j++) {
        if (arr[i] + arr[j] == target) {
            System.out.println(i + " " + j);
        }
    }
}
```

Time complexity: `O(n²)`.

### Approach 2: Optimize using HashMap

The key idea is:

```
complement = target - current number
```

If the complement has already appeared, you have found the answer.

### Java solution

```
import java.util.HashMap;
import java.util.Arrays;

public class Main {

    public static int[] twoSum(int[] arr, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < arr.length; i++) {
            int complement = target - arr[i];

            if (map.containsKey(complement)) {
                return new int[]{map.get(complement), i};
            }

            map.put(arr[i], i);
        }

        return new int[]{-1, -1};
    }

    public static void main(String[] args) {
        int[] arr = {2, 7, 11, 15};
        int target = 9;

        System.out.println(Arrays.toString(twoSum(arr, target)));
    }
}
```

Output:

```
[0, 1]
```

### Dry run

|Current number|Complement|Map before lookup|Result|
|---|---|---|---|
|2|7|`{}`|Store `2 → 0`|
|7|2|`{2=0}`|Found! Return `[0, 1]`|

**Time complexity:** `O(n)` average.

**Space complexity:** `O(n)`.

### Why is this important?

This problem teaches a pattern used in many other interview questions:

1. Calculate the value you need.
2. Look for it in a hash-based data structure.
3. If it exists, use it.
4. Otherwise, store the current element.

---

# 7. Use case 4: Check whether two strings are anagrams

**Problem:** Two strings are anagrams if they contain the same characters with the same frequencies.

Example:

```
s = "listen"
t = "silent"
```

Output:

```
true
```

Another example:

```
s = "hello"
t = "world"
```

Output:

```
false
```

### Java solution using HashMap

```
import java.util.HashMap;

public class Main {

    public static boolean isAnagram(String s, String t) {

        if (s.length() != t.length()) {
            return false;
        }

        HashMap<Character, Integer> map = new HashMap<>();

        for (char c : s.toCharArray()) {
            map.put(c, map.getOrDefault(c, 0) + 1);
        }

        for (char c : t.toCharArray()) {
            if (!map.containsKey(c)) {
                return false;
            }

            int count = map.get(c);

            if (count == 1) {
                map.remove(c);
            } else {
                map.put(c, count - 1);
            }
        }

        return map.isEmpty();
    }

    public static void main(String[] args) {
        System.out.println(isAnagram("listen", "silent")); // true
        System.out.println(isAnagram("hello", "world"));  // false
    }
}
```

### Why does it work?

For `"listen"`, the map initially contains:

```
l -> 1
i -> 1
s -> 1
t -> 1
e -> 1
n -> 1
```

As we process `"silent"`, each character's count decreases.

If a character is missing or occurs too many times, the function returns `false`.

**Time complexity:** `O(n)` average.

**Space complexity:** `O(k)`, where `k` is the number of distinct characters.

This implementation compares Java `char` values. For unrestricted Unicode text, character handling may require a code-point-based approach.

---

# 8. Use case 5: Find the first non-repeating character

**Problem:** Given a string, return its first character that appears only once.

Input:

```
"swiss"
```

Output:

```
w
```

Why? `w` appears once and occurs before `i`, the other non-repeating character.

### Java solution

```
import java.util.HashMap;
import java.util.Map;

public class Main {

    public static Character firstNonRepeating(String s) {
        HashMap<Character, Integer> freq = new HashMap<>();

        for (char c : s.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }

        for (char c : s.toCharArray()) {
            if (freq.get(c) == 1) {
                return c;
            }
        }

        return null;
    }

    public static void main(String[] args) {
        System.out.println(firstNonRepeating("swiss")); // w
        System.out.println(firstNonRepeating("aabb"));  // null
    }
}
```

### Why do we traverse twice?

- First pass: Count the occurrences of every character.
- Second pass: Find the first character whose count is `1`.

A `HashMap` does not guarantee insertion order, so the second pass through the original string preserves the required character order.

**Time complexity:** `O(n)` average.

**Space complexity:** `O(k)`, where `k` is the number of distinct characters.

---

# 9. Use case 6: Find the intersection of two arrays

**Problem:** Find the distinct elements common to both arrays.

Input:

```
arr1 = [1, 2, 2, 3, 4]
arr2 = [2, 3, 5]
```

Output:

```
[2, 3]
```

### Java solution

```
import java.util.HashSet;
import java.util.Set;

public class Main {

    public static Set<Integer> intersection(int[] arr1, int[] arr2) {
        HashSet<Integer> set = new HashSet<>();

        for (int num : arr1) {
            set.add(num);
        }

        HashSet<Integer> result = new HashSet<>();

        for (int num : arr2) {
            if (set.contains(num)) {
                result.add(num);
            }
        }

        return result;
    }

    public static void main(String[] args) {
        int[] arr1 = {1, 2, 2, 3, 4};
        int[] arr2 = {2, 3, 5};

        System.out.println(intersection(arr1, arr2));
    }
}
```

Possible output:

```
[2, 3]
```

The order is not guaranteed.

**Time complexity:** `O(n + m)` average.

**Space complexity:** `O(n + m)` in the worst case, including the result set.

**Use cases:** Comparing user IDs, matching records, finding common elements, and eliminating repeated matches.

---

# 10. Use case 7: Count the frequency of words

**Problem:** Count how many times each word appears in a sentence.

Input:

```
"java is easy and java is powerful"
```

Output:

```
java -> 2
is -> 2
easy -> 1
and -> 1
powerful -> 1
```

### Java solution

```
import java.util.HashMap;
import java.util.Map;

public class Main {

    public static void main(String[] args) {
        String sentence = "java is easy and java is powerful";

        String[] words = sentence.split("\\s+");

        HashMap<String, Integer> freq = new HashMap<>();

        for (String word : words) {
            freq.put(word, freq.getOrDefault(word, 0) + 1);
        }

        for (Map.Entry<String, Integer> entry : freq.entrySet()) {
            System.out.println(
                entry.getKey() + " -> " + entry.getValue()
            );
        }
    }
}
```

**Time complexity:** `O(n)` average in terms of the total number of characters, assuming ordinary bounded-cost word hashing; the exact cost depends on word lengths and hashing operations.

**Use cases:**

- Log analysis.
- Word counting.
- Search engines.
- Text analytics.
- Counting repeated events.

This example treats words as case-sensitive and splits on whitespace. You can normalize case and punctuation if your problem requires it.

---

# 11. How does hashing work internally in Java?

This is useful for interviews, particularly when an interviewer asks about the internal working of `HashMap`.

Suppose you write:

```
HashMap<Integer, String> map = new HashMap<>();

map.put(10, "Apple");
map.put(20, "Banana");
```

Conceptually, Java uses a hash derived from the key to determine where the entry should be stored in its internal table.

The process is approximately:

1. Call the key's `hashCode()` method.
2. Transform the hash as needed.
3. Use the hash to determine a table bucket.
4. Handle collisions if multiple keys map to the same bucket.
5. Use `equals()` to distinguish keys that share a bucket.

### What is a collision?

A collision occurs when different keys map to the same bucket.

For example, two different keys may have the same hash code, or their hash codes may map to the same table index.

Java's `HashMap` handles collisions using bucket-based structures. In suitable circumstances, heavily populated buckets can be converted from linked structures to balanced trees.

**Important:** Different keys can have the same hash code. A hash code is not a guarantee of uniqueness.

### What is the role of `equals()`?

Java uses `hashCode()` to help locate a bucket and `equals()` to check whether keys are equal.

The contract is:

- If two objects are equal according to `equals()`, they must have the same hash code.
- Two objects with the same hash code do not necessarily have to be equal.

This matters when you use custom objects as keys in a `HashMap` or elements in a `HashSet`.

---

# 12. HashMap vs HashSet: Interview comparison

|Feature|HashMap|HashSet|
|---|---|---|
|Stores|Key-value pairs|Unique elements|
|Duplicate keys/elements|Duplicate keys update values|Duplicate elements are ignored|
|Main operation|`put(key, value)`|`add(value)`|
|Lookup|`get(key)`, `containsKey(key)`|`contains(value)`|
|Typical use case|Frequency counting|Duplicate detection|
|Average lookup|`O(1)`|`O(1)`|

These average-time complexities assume reasonable hash distribution. Some operations can take longer in unfavorable circumstances.

### How to choose between them

Use `HashMap` when you need to associate information with a key.

Example:

```
frequency.put(number, count);
```

Use `HashSet` when you only need to track whether an element has appeared.

Example:

```
if (set.contains(number)) {
    // The number has appeared before.
}
```

---

# 13. Top 10 hashing questions for Java interviews

Practice these problems after understanding the examples above.

|No.|Problem|Main concept|Difficulty|
|---|---|---|---|
|1|Contains Duplicate|HashSet|Easy|
|2|Two Sum|HashMap|Easy|
|3|Valid Anagram|Frequency counting|Easy|
|4|First Unique Character in a String|HashMap|Easy|
|5|Intersection of Two Arrays|HashSet|Easy|
|6|Group Anagrams|HashMap and grouping|Medium|
|7|Top K Frequent Elements|HashMap and heap|Medium|
|8|Subarray Sum Equals K|Prefix sum and HashMap|Medium|
|9|Longest Consecutive Sequence|HashSet|Medium|
|10|Longest Substring Without Repeating Characters|HashSet or HashMap|Medium|

Practice links:

- [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)
- [Two Sum](https://leetcode.com/problems/two-sum/)
- [Valid Anagram](https://leetcode.com/problems/valid-anagram/)
- [First Unique Character in a String](https://leetcode.com/problems/first-unique-character-in-a-string/)
- [Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)
- [Group Anagrams](https://leetcode.com/problems/group-anagrams/)
- [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
- [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)
- [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)
- [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

## 14. The most important pattern to remember

Whenever you see a DSA question involving any of the following, consider hashing:

- Finding duplicates.
- Counting occurrences.
- Checking whether something exists.
- Finding pairs that satisfy a condition.
- Grouping elements.
- Tracking previously seen values.
- Calculating subarray sums using prefix sums.

For example, if a brute-force solution takes `O(n²)`, ask yourself:

**"Can I store previously seen information in a HashMap or HashSet so I don't need to scan the array repeatedly?"**

That question often leads to an `O(n)` average-time solution.

### Quick practice question

Try solving this without looking at the answer:

**Given an array, find the first element that appears twice.**

Example:

```
Input:  [4, 2, 7, 2, 4, 8]
Output: 2
```

Hint: Use a `HashSet` and traverse the array from left to right. The first number that is already present in the set is the answer.

This is a good next exercise because it tests whether you understand how hashing works rather than just memorizing the syntax.