# All Important Array Algorithms in Java

Array algorithms are essential for Java programming, coding interviews, DSA (Data Structures and Algorithms), and problem-solving.

Below is a comprehensive list of array algorithms, organized by category, from **basic to advanced**, with Java-oriented examples and the key problems you should practice.

## 1\. Basic Array Algorithms

These are the fundamental operations you should learn first.

| No. | Algorithm                | Description                                       |
| --- | ------------------------ | ------------------------------------------------- |
| 1   | Array Traversal          | Visit every element in an array                   |
| 2   | Array Insertion          | Insert an element at a specific position          |
| 3   | Array Deletion           | Remove an element from an array                   |
| 4   | Array Updation           | Update an element at a specific index             |
| 5   | Array Searching          | Find an element in an array                       |
| 6   | Array Copying            | Copy elements from one array to another           |
| 7   | Array Reversal           | Reverse the order of elements                     |
| 8   | Array Rotation           | Rotate elements left or right                     |
| 9   | Array Merging            | Combine two arrays                                |
| 10  | Array Concatenation      | Append one array to another                       |
| 11  | Array Cloning            | Create a copy using `clone()`                     |
| 12  | Array Comparison         | Compare two arrays                                |
| 13  | Array Sorting            | Arrange elements in ascending or descending order |
| 14  | Find Array Length        | Determine the number of elements                  |
| 15  | Find Minimum and Maximum | Find the smallest and largest elements            |

Example:

```
int[] arr = {10, 20, 30, 40, 50};

// Traversal
for (int num : arr) {
    System.out.println(num);
}

// Find minimum and maximum
int min = arr[0];
int max = arr[0];

for (int num : arr) {
    min = Math.min(min, num);
    max = Math.max(max, num);
}

// Reverse
int left = 0;
int right = arr.length - 1;

while (left < right) {
    int temp = arr[left];
    arr[left] = arr[right];
    arr[right] = temp;

    left++;
    right--;
}
```

---

## 2\. Searching Algorithms

Searching algorithms help locate elements in arrays.

| No. | Algorithm                            | Time Complexity | Requirement                    |
| --- | ------------------------------------ | --------------- | ------------------------------ |
| 1   | Linear Search                        | O(n)            | Any array                      |
| 2   | Binary Search                        | O(log n)        | Sorted array                   |
| 3   | Recursive Binary Search              | O(log n)        | Sorted array                   |
| 4   | Lower Bound                          | O(log n)        | Sorted array                   |
| 5   | Upper Bound                          | O(log n)        | Sorted array                   |
| 6   | First Occurrence                     | O(log n)        | Sorted array                   |
| 7   | Last Occurrence                      | O(log n)        | Sorted array                   |
| 8   | Count Occurrences                    | O(log n)        | Sorted array                   |
| 9   | Search Insert Position               | O(log n)        | Sorted array                   |
| 10  | Search in Rotated Sorted Array       | O(log n)        | Rotated sorted array           |
| 11  | Find Minimum in Rotated Sorted Array | O(log n)        | Rotated sorted array           |
| 12  | Peak Element Search                  | O(log n)        | Suitable array structure       |
| 13  | Search in Nearly Sorted Array        | O(log n)        | Nearly sorted array            |
| 14  | Exponential Search                   | O(log n)        | Sorted array                   |
| 15  | Ternary Search                       | O(log n)        | Suitable unimodal search space |

### Important problems to practice

- Find an element in an array.
- Find the first and last occurrence of a target.
- Count the frequency of a target.
- Find the insertion position of an element.
- Search in a rotated sorted array.
- Find the minimum element in a rotated sorted array.
- Find a peak element.
- Find the square root of a number using binary search.
- Find the single element in a sorted array where other elements appear twice.

**Interview tip:** Master binary search and its variations. Many advanced array problems use binary search on an answer rather than searching directly for an element.

---

## 3\. Sorting Algorithms

Sorting is one of the most important topics in DSA.

| No. | Algorithm      | Best Case               | Average Case            | Worst Case              |
| --- | -------------- | ----------------------- | ----------------------- | ----------------------- |
| 1   | Bubble Sort    | O(n)                    | O(n²)                   | O(n²)                   |
| 2   | Selection Sort | O(n²)                   | O(n²)                   | O(n²)                   |
| 3   | Insertion Sort | O(n)                    | O(n²)                   | O(n²)                   |
| 4   | Merge Sort     | O(n log n)              | O(n log n)              | O(n log n)              |
| 5   | Quick Sort     | O(n log n)              | O(n log n)              | O(n²)                   |
| 6   | Heap Sort      | O(n log n)              | O(n log n)              | O(n log n)              |
| 7   | Counting Sort  | O(n + k)                | O(n + k)                | O(n + k)                |
| 8   | Radix Sort     | O(d(n + b))             | O(d(n + b))             | O(d(n + b))             |
| 9   | Bucket Sort    | O(n + k)                | Depends on distribution | O(n²)                   |
| 10  | Shell Sort     | Depends on gap sequence | Depends on gap sequence | Depends on gap sequence |

Here:

- `n` = number of elements.
- `k` = value range or number of buckets, depending on the algorithm.
- `d` = number of digits.
- `b` = radix or digit base.

### Important sorting problems

- Sort an array in ascending order.
- Sort an array containing only `0`, `1`, and `2`.
- Merge two sorted arrays.
- Merge overlapping intervals.
- Find the kth largest element.
- Find the kth smallest element.
- Sort an array according to element frequency.
- Count inversions in an array.
- Find the minimum number of swaps required to sort an array.

In Java, you should also understand:

```
Arrays.sort(arr);
```

For primitive arrays, Java uses an optimized implementation rather than requiring you to implement a sorting algorithm manually.

---

## 4\. Two-Pointer Algorithms

The two-pointer technique uses two indices to solve problems efficiently, often in O(n) time.

| No. | Algorithm / Pattern | Typical Use |
| --- | --- | --- |
| 1 | Opposite-Direction Pointers | Pair sums in sorted arrays |
| 2 | Same-Direction Pointers | Array compaction |
| 3 | Fast and Slow Pointers | Cycle detection and duplicate-related problems |
| 4 | Read and Write Pointers | In-place modifications |
| 5 | Two-Pointer Pair Sum | Find two numbers with a target sum |
| 6 | Three-Pointer Technique | Problems involving three sequences or partitions |
| 7 | Dutch National Flag Algorithm | Sort `0`, `1`, and `2` |
| 8 | Remove Duplicates In-Place | Remove duplicates from sorted arrays |
| 9 | Move Zeroes | Move zeroes to the end |
| 10 | Partition Array | Separate elements by a condition |
| 11 | Container With Most Water | Maximize the area between two boundaries |
| 12 | Trapping Rain Water | Calculate trapped water using two pointers |
| 13 | Three Sum | Find triplets with a target sum |
| 14 | Four Sum | Find quadruplets with a target sum |
| 15 | Merge Sorted Arrays In-Place | Merge sorted arrays using pointers |

### Example: Move zeroes to the end

Input:

```
int[] arr = {0, 1, 0, 3, 12};
```

Output:

```
[1, 3, 12, 0, 0]
```

Java solution:

```
public static void moveZeroes(int[] arr) {
    int index = 0;

    for (int num : arr) {
        if (num != 0) {
            arr[index++] = num;
        }
    }

    while (index < arr.length) {
        arr[index++] = 0;
    }
}
```

- Time complexity: O(n).
- Extra space complexity: O(1).

---

## 5\. Sliding Window Algorithms

Sliding window is particularly useful for contiguous subarray problems.

There are two main types:

- **Fixed-size window:** The window length remains constant.
- **Variable-size window:** The window expands or shrinks based on a condition.

| No. | Algorithm / Pattern | Typical Problem |
| --- | --- | --- |
| 1 | Fixed-Size Sliding Window | Maximum sum of a subarray of size `k` |
| 2 | Variable-Size Sliding Window | Smallest subarray satisfying a condition |
| 3 | Maximum Sum Subarray of Size K | Find the maximum sum for a fixed window |
| 4 | Minimum Sum Subarray of Size K | Find the minimum sum for a fixed window |
| 5 | Maximum Average Subarray | Find the maximum average for a fixed window |
| 6 | Longest Subarray With Sum at Most K | Find the longest valid window under suitable constraints |
| 7 | Longest Subarray With At Most K Distinct Elements | Maintain a frequency map |
| 8 | Minimum Window Subarray | Find the smallest window satisfying a condition |
| 9 | Count Subarrays With Product Less Than K | Positive-number sliding window |
| 10 | Maximum Consecutive Ones | Find the longest run of ones |
| 11 | Maximum Consecutive Ones III | Find the longest run after flipping at most K zeroes |
| 12 | Sliding Window Maximum | Find the maximum for every window |
| 13 | Sliding Window Minimum | Find the minimum for every window |
| 14 | Longest Subarray After Replacing Elements | Maintain a window under a replacement limit |
| 15 | Subarrays With Exactly K Distinct Elements | Combine counts of at-most-K windows |

### Example: Maximum sum of a subarray of size K

Input:

```
int[] arr = {2, 1, 5, 1, 3, 2};
int k = 3;
```

Output:

```
9
```

The subarray `{5, 1, 3}` has the maximum sum of `9`.

Java solution:

```
public static int maxSumSubarray(int[] arr, int k) {
    if (k <= 0 || k > arr.length) {
        throw new IllegalArgumentException("Invalid window size");
    }

    int windowSum = 0;

    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }

    int maxSum = windowSum;

    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}
```

Time complexity: O(n).

**Important:** Sliding window does not work for every subarray-sum problem. For example, negative numbers can invalidate common sum-based window-shrinking logic. Prefix sums or other techniques may be more appropriate.

---

## 6\. Prefix Sum and Suffix Sum Algorithms

Prefix and suffix sums allow you to answer certain range and subarray queries efficiently.

| No. | Algorithm | Typical Use |
| --- | --- | --- |
| 1 | Prefix Sum | Sum of elements from the beginning to an index |
| 2 | Suffix Sum | Sum of elements from an index to the end |
| 3 | Range Sum Query | Calculate sums over an interval |
| 4 | Prefix Product | Product of elements up to an index |
| 5 | Suffix Product | Product of elements from an index to the end |
| 6 | Equilibrium Index | Equal sum on the left and right |
| 7 | Pivot Index | Find an index where left and right sums match |
| 8 | Product of Array Except Self | Calculate products without using the current element |
| 9 | Prefix XOR | Answer range XOR queries |
| 10 | Difference Array | Perform efficient range updates |
| 11 | Prefix Frequency | Count occurrences in ranges |
| 12 | 2D Prefix Sum | Answer matrix rectangle-sum queries |

### Example: Prefix sum

Input:

```
int[] arr = {2, 4, 6, 8, 10};
```

Prefix sum:

```
[2, 6, 12, 20, 30]
```

Java solution:

```
public static int[] prefixSum(int[] arr) {
    int[] prefix = new int[arr.length];

    if (arr.length == 0) {
        return prefix;
    }

    prefix[0] = arr[0];

    for (int i = 1; i < arr.length; i++) {
        prefix[i] = prefix[i - 1] + arr[i];
    }

    return prefix;
}
```

After building the prefix sum, the sum from index `left` to index `right`, inclusive, is:

```
int sum = prefix[right]
        - (left > 0 ? prefix[left - 1] : 0);
```

Preprocessing takes O(n), and each range-sum query takes O(1).

---

## 7\. Subarray Algorithms

A subarray is a **contiguous portion of an array**.

Subarray problems are frequently asked in technical interviews.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Generate All Subarrays | Enumerate every contiguous subarray |
| 2 | Maximum Subarray Sum | Find the maximum-sum subarray |
| 3 | Minimum Subarray Sum | Find the minimum-sum subarray |
| 4 | Maximum Product Subarray | Find the maximum product of a contiguous subarray |
| 5 | Subarray Sum Equals K | Count subarrays with sum K |
| 6 | Longest Subarray With Sum K | Find the longest subarray with a given sum |
| 7 | Shortest Subarray With Sum at Least K | Find the shortest qualifying subarray |
| 8 | Count Subarrays With Sum K | Count matching subarrays |
| 9 | Zero-Sum Subarray | Determine whether a zero-sum subarray exists |
| 10 | Longest Zero-Sum Subarray | Find the longest zero-sum subarray |
| 11 | Count Subarrays With XOR K | Count subarrays whose XOR equals K |
| 12 | Maximum Circular Subarray Sum | Find the maximum sum in a circular array |
| 13 | Maximum Sum of Two Non-Overlapping Subarrays | Optimize two disjoint subarrays |
| 14 | Maximum Sum of Three Non-Overlapping Subarrays | Optimize three disjoint subarrays |
| 15 | Count Subarrays With Equal Zeroes and Ones | Use a prefix transformation |
| 16 | Count Subarrays Divisible by K | Use prefix remainders |
| 17 | Maximum Alternating Subarray Sum | Maximize an alternating-sign sum |
| 18 | Maximum Subarray Sum With One Deletion | Allow one element to be removed |

### Kadane's Algorithm

Kadane's algorithm finds the maximum sum of a non-empty contiguous subarray in O(n) time.

Example:

```
int[] arr = {-2, 1, -3, 4, -1, 2, 1, -5, 4};
```

Output:

```
6
```

The maximum-sum subarray is:

```
[4, -1, 2, 1]
```

Java implementation:

```
public static int maxSubarraySum(int[] arr) {
    if (arr.length == 0) {
        throw new IllegalArgumentException("Array must not be empty");
    }

    int currentSum = arr[0];
    int maxSum = arr[0];

    for (int i = 1; i < arr.length; i++) {
        currentSum = Math.max(arr[i], currentSum + arr[i]);
        maxSum = Math.max(maxSum, currentSum);
    }

    return maxSum;
}
```

Time complexity: O(n).

Space complexity: O(1).

This implementation also handles arrays containing only negative numbers.

---

## 8\. Hashing and Frequency-Based Algorithms

Hashing helps solve many array problems efficiently using `HashMap` and `HashSet`.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Frequency Counting | Count the occurrences of each element |
| 2 | Find Duplicates | Identify repeated elements |
| 3 | Find Unique Elements | Find elements that occur once |
| 4 | Find Missing Number | Find a missing number in a range |
| 5 | Find All Missing Numbers | Find all absent values in a specified range |
| 6 | Find Repeating Number | Identify repeated values |
| 7 | Find the First Repeating Element | Find the earliest element that repeats |
| 8 | Find the First Non-Repeating Element | Find the earliest element occurring once |
| 9 | Two Sum | Find a pair with a target sum |
| 10 | Count Pairs With Sum K | Count pairs whose sum is K |
| 11 | Count Pairs With Difference K | Count pairs with a given difference |
| 12 | Longest Consecutive Sequence | Find the longest consecutive-value sequence |
| 13 | Majority Element | Find an element occurring more than n/2 times |
| 14 | Majority Elements II | Find elements occurring more than n/3 times |
| 15 | Intersection of Arrays | Find common elements |
| 16 | Union of Arrays | Find distinct elements from both arrays |
| 17 | Find Two Numbers With XOR K | Use hashing or frequency counting |
| 18 | Group Elements by Frequency | Organize elements by occurrence count |

### Example: Two Sum

Input:

```
int[] arr = {2, 7, 11, 15};
int target = 9;
```

Output:

```
[0, 1]
```

Java solution:

```
import java.util.HashMap;
import java.util.Map;

public static int[] twoSum(int[] arr, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < arr.length; i++) {
        int complement = target - arr[i];

        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }

        map.put(arr[i], i);
    }

    return new int[]{-1, -1};
}
```

- Time complexity: O(n) on average.
- Extra space complexity: O(n).

---

## 9\. In-Place Array Algorithms

In-place algorithms modify the existing array while using little or no additional space.

| No. | Algorithm | Typical Use |
| --- | --- | --- |
| 1 | Reverse Array In-Place | Reverse elements |
| 2 | Left Rotate by One | Move the first element to the end |
| 3 | Right Rotate by One | Move the last element to the beginning |
| 4 | Rotate Array by K Positions | Rotate by multiple positions |
| 5 | Remove Duplicates In-Place | Compact unique elements |
| 6 | Remove a Specific Element | Remove all occurrences of a value |
| 7 | Move Zeroes | Move zeroes to the end |
| 8 | Move Negative Numbers to One Side | Partition negative and non-negative values |
| 9 | Segregate Even and Odd Numbers | Partition by parity |
| 10 | Rearrange Positive and Negative Numbers | Alternate or separate signs |
| 11 | Next Permutation | Find the next lexicographically greater permutation |
| 12 | Previous Permutation | Find the previous lexicographic permutation |
| 13 | Cyclic Sort | Place values in their corresponding positions |
| 14 | Find Missing Number Using Index Placement | Rearrange values to identify missing elements |
| 15 | Find Duplicate Using Index Placement | Detect repeated values under suitable constraints |

### Example: Rotate an array to the right by K positions

Input:

```
Array: [1, 2, 3, 4, 5, 6, 7]
K = 3
```

Output:

```
[5, 6, 7, 1, 2, 3, 4]
```

A common O(n)-time, O(1)-extra-space solution uses three reversals:

1. Reverse the entire array.
2. Reverse the first K elements.
3. Reverse the remaining elements.

Remember to normalize K using `k %= arr.length` when the array is non-empty.

---

## 10\. Greedy Array Algorithms

Greedy algorithms make locally optimal decisions to solve problems that have suitable greedy properties.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Best Time to Buy and Sell Stock | Maximize profit from one transaction |
| 2 | Stock Buy and Sell II | Maximize profit with multiple transactions |
| 3 | Jump Game | Determine whether the last index is reachable |
| 4 | Jump Game II | Find the minimum jumps needed |
| 5 | Gas Station | Find a valid circular starting station |
| 6 | Candy Distribution | Assign candies under neighboring constraints |
| 7 | Minimum Platforms | Find the minimum platforms needed for trains |
| 8 | Activity Selection | Select the maximum number of non-overlapping activities |
| 9 | Interval Scheduling | Choose compatible intervals |
| 10 | Minimum Number of Arrows | Burst all overlapping intervals |
| 11 | Assign Cookies | Maximize the number of satisfied children |
| 12 | Job Sequencing | Schedule jobs to maximize profit under constraints |
| 13 | Partition Labels | Partition a string into independent segments |
| 14 | Minimum Number of Swaps | Solve suitable ordering and grouping problems |

Note that some of these problems use arrays as input but are not exclusively array algorithms. For example, interval scheduling is fundamentally a greedy problem that often uses sorting.

---

## 11\. Binary Search on the Answer

In this technique, binary search is performed over a possible answer rather than directly over array indices.

| No. | Algorithm / Problem | Objective |
| --- | --- | --- |
| 1 | Allocate Books | Minimize the maximum pages assigned |
| 2 | Split Array Largest Sum | Minimize the largest partition sum |
| 3 | Painter's Partition | Minimize the maximum workload |
| 4 | Aggressive Cows | Maximize the minimum distance |
| 5 | Koko Eating Bananas | Find the minimum eating speed |
| 6 | Minimum Days to Make Bouquets | Find the minimum feasible day |
| 7 | Capacity to Ship Packages | Find the minimum shipping capacity |
| 8 | Smallest Divisor Given a Threshold | Find the smallest valid divisor |
| 9 | Median of Two Sorted Arrays | Use binary partitioning |
| 10 | Find Kth Element of Two Sorted Arrays | Find an element by partitioning |

These problems are especially useful for intermediate and advanced DSA preparation.

---

## 12\. Monotonic Stack and Monotonic Deque Algorithms

These techniques maintain elements in increasing or decreasing order to answer range-related questions efficiently.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Next Greater Element | Find the next greater element for each value |
| 2 | Next Smaller Element | Find the next smaller element |
| 3 | Previous Greater Element | Find the previous greater element |
| 4 | Previous Smaller Element | Find the previous smaller element |
| 5 | Daily Temperatures | Find days until a warmer temperature |
| 6 | Stock Span | Calculate the stock span for each day |
| 7 | Largest Rectangle in Histogram | Find the maximum rectangle area |
| 8 | Maximal Rectangle | Find the largest rectangle of ones in a matrix |
| 9 | Sliding Window Maximum | Maintain maximum values using a deque |
| 10 | Sum of Subarray Minimums | Calculate the sum of minimum values across subarrays |
| 11 | Sum of Subarray Ranges | Sum the maximum-minus-minimum value of each subarray |
| 12 | Trapping Rain Water | Solve using a stack or two pointers |

A common time complexity for these algorithms is O(n), because each element is generally pushed and popped at most once.

---

## 13\. Bit Manipulation Algorithms for Arrays

Bit manipulation is useful when array values have particular properties.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Find the Single Number | Every other value appears twice |
| 2 | Find Two Unique Numbers | All other values appear twice |
| 3 | Find the Missing Number Using XOR | Find a missing value in a range |
| 4 | Count Set Bits | Count the number of one-bits |
| 5 | Check Power of Two | Test a number using bit operations |
| 6 | XOR of All Array Elements | Calculate cumulative XOR |
| 7 | Find a Number Appearing Once | Solve suitable frequency constraints |
| 8 | Count Bits for All Numbers | Calculate bit counts for a range |
| 9 | Find the Maximum XOR of Two Elements | Use a binary trie or bitwise greedy method |
| 10 | Count Subarrays With XOR K | Combine prefix XOR and hashing |

### Example: Find the single number

Input:

```
[4, 1, 2, 1, 2]
```

Output:

```
4
```

Java implementation:

```
public static int singleNumber(int[] arr) {
    int result = 0;

    for (int num : arr) {
        result ^= num;
    }

    return result;
}
```

This works when exactly one element occurs once and every other element occurs exactly twice.

Time complexity: O(n).

Extra space complexity: O(1).

---

## 14\. Divide and Conquer Array Algorithms

Divide and conquer breaks a problem into smaller subproblems and combines their results.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Merge Sort | Sort an array |
| 2 | Quick Sort | Sort an array using partitioning |
| 3 | Binary Search | Search a sorted array |
| 4 | Count Inversions | Count out-of-order pairs |
| 5 | Maximum Subarray Using Divide and Conquer | Find the maximum-sum subarray |
| 6 | Find the Majority Element | Use recursive partitioning or another suitable approach |
| 7 | Median of Two Sorted Arrays | Find a median using partitioning |
| 8 | Kth Smallest Element | Use selection algorithms such as Quickselect |
| 9 | Closest Pair of Points | Find the closest pair in a point set |
| 10 | Selection Algorithm | Find the kth order statistic |

Some of these algorithms are commonly taught as divide-and-conquer techniques, while others have alternative solutions using different paradigms.

---

## 15\. Dynamic Programming Algorithms on Arrays

Dynamic programming is used when a problem can be broken into overlapping subproblems with reusable results.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Maximum Subarray Sum | Kadane's algorithm is a space-optimized DP solution |
| 2 | Maximum Product Subarray | Track maximum and minimum products |
| 3 | Longest Increasing Subsequence (LIS) | Find the longest strictly increasing subsequence |
| 4 | Longest Decreasing Subsequence | Find the longest decreasing subsequence |
| 5 | Longest Bitonic Subsequence | Increasing followed by decreasing |
| 6 | Maximum Sum Increasing Subsequence | Maximize the sum of an increasing subsequence |
| 7 | House Robber | Maximize non-adjacent selections |
| 8 | House Robber II | Handle circular arrangements |
| 9 | Partition Equal Subset Sum | Determine whether an equal partition is possible |
| 10 | Subset Sum | Determine whether a target sum is possible |
| 11 | Minimum Subset Sum Difference | Minimize the difference between subset sums |
| 12 | Coin Change | Find the minimum coins or number of ways |
| 13 | 0/1 Knapsack | Maximize value under a weight constraint |
| 14 | Unbounded Knapsack | Allow repeated item selection |
| 15 | Maximum Alternating Subsequence Sum | Maximize an alternating-sign subsequence |
| 16 | Maximum Non-Adjacent Sum | Select elements without choosing adjacent positions |
| 17 | Minimum Cost Climbing Stairs | Minimize the total climbing cost |
| 18 | Number of Longest Increasing Subsequences | Count longest increasing subsequences |

### Important distinction

A **subarray** contains contiguous elements, whereas a **subsequence** preserves relative order but does not have to be contiguous.

For example, in:

```
[1, 5, 2, 6]
```

- `[5, 2]` is a subarray.
- `[1, 2, 6]` is a subsequence, but not a subarray.

---

## 16\. Advanced Array Algorithms

These problems are useful for advanced interviews and competitive programming.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Boyer–Moore Voting Algorithm | Find a majority element |
| 2 | Extended Boyer–Moore Algorithm | Find elements appearing more than n/3 times |
| 3 | Quickselect | Find the kth smallest or largest element |
| 4 | Floyd's Cycle Detection | Find a duplicate under specific array constraints |
| 5 | Mo's Algorithm | Answer offline range queries efficiently |
| 6 | Segment Tree | Support range queries and updates |
| 7 | Fenwick Tree (Binary Indexed Tree) | Support prefix queries and point updates |
| 8 | Sparse Table | Answer static range-minimum or similar queries |
| 9 | Difference Array | Perform efficient range updates |
| 10 | Coordinate Compression | Map values to compact ranks |
| 11 | Inversion Counting | Count pairs where earlier elements exceed later elements |
| 12 | Count Smaller Elements After Self | Use merge sort or an indexed data structure |
| 13 | Maximum XOR Pair | Find two elements with the maximum XOR |
| 14 | Median Maintenance | Maintain a running median using two heaps |
| 15 | Range Minimum Query (RMQ) | Find the minimum within an interval |
| 16 | Range Maximum Query | Find the maximum within an interval |
| 17 | Kth Smallest in a Matrix | Find an order statistic in a sorted matrix |
| 18 | Maximum Subarray XOR | Find a subarray with maximum XOR |
| 19 | Persistent Segment Tree | Answer versioned range queries |
| 20 | Lazy Propagation | Optimize range updates in segment trees |

---

## 17\. Matrix and Two-Dimensional Array Algorithms

Java supports multidimensional arrays, so matrix algorithms are also important.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Matrix Traversal | Visit every matrix element |
| 2 | Row-Wise Traversal | Process each row |
| 3 | Column-Wise Traversal | Process each column |
| 4 | Transpose Matrix | Swap rows and columns |
| 5 | Rotate Matrix by 90 Degrees | Rotate a square matrix |
| 6 | Spiral Traversal | Traverse the matrix in spiral order |
| 7 | Boundary Traversal | Visit boundary elements |
| 8 | Diagonal Traversal | Traverse diagonals |
| 9 | Search in a Sorted Matrix | Find a target efficiently |
| 10 | Set Matrix Zeroes | Zero rows and columns based on zero-valued elements |
| 11 | Matrix Multiplication | Multiply compatible matrices |
| 12 | Matrix Addition | Add corresponding elements |
| 13 | Prefix Sum Matrix | Answer rectangle-sum queries |
| 14 | Maximum Sum Rectangle | Find a maximum-sum submatrix |
| 15 | Maximum Rectangle of Ones | Find the largest all-ones rectangle |
| 16 | Flood Fill | Replace connected regions |
| 17 | Number of Islands | Count connected land regions |
| 18 | Shortest Path in a Grid | Find a path through a matrix |
| 19 | Search a Row-Wise and Column-Wise Sorted Matrix | Search using matrix ordering |
| 20 | Diagonal Sum | Calculate primary and secondary diagonal sums |

---

## 18\. Permutation and Rearrangement Algorithms

These algorithms rearrange array elements according to specific ordering rules.

| No. | Algorithm | Typical Problem |
| --- | --- | --- |
| 1 | Next Permutation | Find the next lexicographically greater arrangement |
| 2 | Previous Permutation | Find the previous lexicographic arrangement |
| 3 | Generate All Permutations | Enumerate every possible arrangement |
| 4 | Generate Unique Permutations | Avoid duplicate arrangements |
| 5 | Cyclic Sort | Place values at corresponding indices |
| 6 | Wiggle Sort | Arrange elements in alternating order |
| 7 | Wave Array | Arrange elements in a wave-like pattern |
| 8 | Alternate Positive and Negative Numbers | Rearrange signs |
| 9 | Sort Colors | Group `0`, `1`, and `2` |
| 10 | Rearrange by Index | Place elements according to an index mapping |
| 11 | Shuffle an Array | Randomize an array |
| 12 | Interleave Two Arrays | Alternate elements from two arrays |
| 13 | Reorder Array by Parity | Group even and odd elements |
| 14 | Reorder Array by Frequency | Arrange values according to occurrence counts |

---

# 19\. Most Important Array Problems for Java Interviews

If you are preparing for Java developer interviews, start with these **30 high-value problems**.

| No. | Problem | Main Technique |
| --- | --- | --- |
| 1 | Find Largest Element | Traversal |
| 2 | Find Second Largest Element | Traversal |
| 3 | Check if Array Is Sorted | Traversal |
| 4 | Reverse an Array | Two pointers |
| 5 | Remove Duplicates From Sorted Array | Two pointers |
| 6 | Rotate Array by K | Reversal |
| 7 | Move Zeroes | Two pointers |
| 8 | Find Missing Number | Mathematics or XOR |
| 9 | Find the Single Number | XOR |
| 10 | Two Sum | HashMap |
| 11 | Sort Colors | Dutch National Flag |
| 12 | Best Time to Buy and Sell Stock | Greedy |
| 13 | Maximum Subarray Sum | Kadane's algorithm |
| 14 | Maximum Product Subarray | Dynamic programming |
| 15 | Majority Element | Boyer–Moore |
| 16 | Majority Elements II | Extended Boyer–Moore |
| 17 | Find Duplicate Number | Floyd's cycle detection |
| 18 | Merge Sorted Arrays | Two pointers |
| 19 | Find the Union of Two Sorted Arrays | Two pointers |
| 20 | Find the Intersection of Two Arrays | Hashing or two pointers |
| 21 | Longest Consecutive Sequence | HashSet |
| 22 | Subarray Sum Equals K | Prefix sum + HashMap |
| 23 | Longest Subarray With Sum K | Prefix sum + HashMap |
| 24 | Three Sum | Sorting + two pointers |
| 25 | Container With Most Water | Two pointers |
| 26 | Trapping Rain Water | Two pointers or stack |
| 27 | Search in Rotated Sorted Array | Binary search |
| 28 | Find the Kth Largest Element | Heap or Quickselect |
| 29 | Next Permutation | Rearrangement |
| 30 | Longest Increasing Subsequence | Dynamic programming or binary search |

These problems cover many of the most frequently used array techniques.

---

# 20\. Recommended Learning Order

Do not try to learn every algorithm at once. Follow this progression.

### Level 1: Beginner

Learn these first:

1. Array traversal.
2. Find minimum and maximum.
3. Find second largest element.
4. Reverse an array.
5. Check whether an array is sorted.
6. Remove duplicates from a sorted array.
7. Left and right rotation.
8. Move zeroes to the end.
9. Linear search.
10. Binary search.

### Level 2: Basic to Intermediate

Next, study:

1. Bubble sort.
2. Selection sort.
3. Insertion sort.
4. Merge two sorted arrays.
5. Two Sum.
6. Frequency counting.
7. Find missing and duplicate numbers.
8. Prefix sums.
9. Two-pointer technique.
10. Sliding window.

### Level 3: Intermediate

Focus on:

1. Kadane's algorithm.
2. Maximum product subarray.
3. Three Sum.
4. Majority Element.
5. Longest Consecutive Sequence.
6. Subarray Sum Equals K.
7. Search in Rotated Sorted Array.
8. Next Permutation.
9. Trapping Rain Water.
10. Container With Most Water.

### Level 4: Advanced

Then move on to:

1. Monotonic stack.
2. Sliding Window Maximum.
3. Merge Sort and inversion counting.
4. Quickselect.
5. Binary Search on Answer.
6. Longest Increasing Subsequence.
7. Segment Tree.
8. Fenwick Tree.
9. Mo's Algorithm.
10. Advanced range-query problems.

---

## 21\. Java Methods You Should Know

Java provides built-in array utilities through the `Arrays` class.

```
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] arr = {5, 2, 8, 1, 3};

        // Sort
        Arrays.sort(arr);

        // Print array
        System.out.println(Arrays.toString(arr));

        // Binary search (array must be sorted)
        int index = Arrays.binarySearch(arr, 3);

        // Copy array
        int[] copy = Arrays.copyOf(arr, arr.length);

        // Compare arrays
        boolean equal = Arrays.equals(arr, copy);

        // Fill array
        int[] numbers = new int[5];
        Arrays.fill(numbers, 10);

        // Print results
        System.out.println("Index: " + index);
        System.out.println("Equal: " + equal);
        System.out.println(Arrays.toString(numbers));
    }
}
```

Important methods include:

- `Arrays.sort()`
- `Arrays.binarySearch()`
- `Arrays.copyOf()`
- `Arrays.equals()`
- `Arrays.fill()`
- `Arrays.toString()`
- `Arrays.deepToString()` for nested arrays
- `Arrays.deepEquals()` for nested arrays

**Remember:** Built-in methods are useful in real Java development, but you should also know how the underlying algorithms work for DSA interviews.

---

## Final Recommendation

If your goal is to prepare for Java coding interviews, prioritize these **10 techniques**:

1. **Two Pointers**
2. **Sliding Window**
3. **Prefix Sum**
4. **Hashing**
5. **Binary Search**
6. **Sorting**
7. **Kadane's Algorithm**
8. **Greedy Algorithms**
9. **Monotonic Stack**
10. **Dynamic Programming**

These techniques help solve a large proportion of common array problems.

The key is not to memorize hundreds of algorithms individually. Instead, **learn each technique, understand when to use it, and practice multiple problems that apply the same pattern.**
