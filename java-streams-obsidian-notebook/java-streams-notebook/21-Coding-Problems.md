---
tags: [java, streams, coding-problems]
---

# 21. 30 Coding Problems with Solutions

⬅ Back to [[00-Home]] | Prev: [[20-Interview-Questions]] | Next: [[22-Real-World-Examples]]

> Assume a `record Employee(String name, String department, double salary, int age, boolean active)` unless stated otherwise. Solutions favor Stream idioms; loop alternatives are noted where instructive.

**1. Find all even numbers in a list.**
```java
List<Integer> evens = nums.stream().filter(n -> n % 2 == 0).toList();
```

**2. Find the sum of all numbers.**
```java
int sum = nums.stream().mapToInt(Integer::intValue).sum();
```

**3. Find the maximum element.**
```java
Optional<Integer> max = nums.stream().max(Integer::compareTo);
```

**4. Convert a list of strings to uppercase.**
```java
List<String> upper = words.stream().map(String::toUpperCase).toList();
```

**5. Count strings starting with a given letter.**
```java
long count = words.stream().filter(w -> w.startsWith("A")).count();
```

**6. Remove duplicates from a list.**
```java
List<Integer> unique = nums.stream().distinct().toList();
```

**7. Sort a list of strings by length, then alphabetically.**
```java
List<String> sorted = words.stream()
        .sorted(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()))
        .toList();
```

**8. Find the first word longer than 5 characters.**
```java
Optional<String> first = words.stream().filter(w -> w.length() > 5).findFirst();
```

**9. Join a list of strings with commas.**
```java
String joined = words.stream().collect(Collectors.joining(", "));
```

**10. Group employees by department.**
```java
Map<String, List<Employee>> byDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::department));
```

**11. Count employees per department.**
```java
Map<String, Long> countByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::department, Collectors.counting()));
```

**12. Find the average salary per department.**
```java
Map<String, Double> avgByDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::department, Collectors.averagingDouble(Employee::salary)));
```

**13. Find the highest-paid employee overall.**
```java
Optional<Employee> topEarner = employees.stream()
        .max(Comparator.comparingDouble(Employee::salary));
```

**14. Find the highest-paid employee per department.**
```java
Map<String, Optional<Employee>> topPerDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::department,
                Collectors.maxBy(Comparator.comparingDouble(Employee::salary))));
```

**15. Partition employees into active vs inactive.**
```java
Map<Boolean, List<Employee>> partitioned = employees.stream()
        .collect(Collectors.partitioningBy(Employee::active));
```

**16. Flatten a list of lists of integers.**
```java
List<Integer> flat = listOfLists.stream().flatMap(List::stream).toList();
```

**17. Find all unique characters in a list of words.**
```java
Set<Character> chars = words.stream()
        .flatMap(w -> w.chars().mapToObj(c -> (char) c))
        .collect(Collectors.toSet());
```

**18. Reverse a list using Streams.**
```java
List<Integer> reversed = IntStream.range(0, nums.size())
        .mapToObj(i -> nums.get(nums.size() - 1 - i))
        .toList();
```

**19. Check if all employees are active.**
```java
boolean allActive = employees.stream().allMatch(Employee::active);
```

**20. Find the second-highest salary.**
```java
Optional<Double> second = employees.stream()
        .map(Employee::salary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst();
```

**21. Convert a list of employees to a `Map<name, salary>`.**
```java
Map<String, Double> nameToSalary = employees.stream()
        .collect(Collectors.toMap(Employee::name, Employee::salary));
```

**22. Find the total salary expense using primitive streams.**
```java
double total = employees.stream().mapToDouble(Employee::salary).sum();
```

**23. Generate the first 10 Fibonacci numbers using `Stream.iterate`.**
```java
List<Long> fib = Stream.iterate(new long[]{0, 1}, f -> new long[]{f[1], f[0] + f[1]})
        .limit(10)
        .map(f -> f[0])
        .toList();
```

**24. Check if a string is a palindrome using IntStream.**
```java
static boolean isPalindrome(String s) {
    return IntStream.range(0, s.length() / 2)
            .allMatch(i -> s.charAt(i) == s.charAt(s.length() - 1 - i));
}
```

**25. Find word frequency in a sentence.**
```java
Map<String, Long> freq = Arrays.stream(sentence.toLowerCase().split("\\s+"))
        .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```

**26. Merge two maps of `String -> Integer`, summing values on key conflict.**
```java
Map<String, Integer> merged = Stream.of(map1, map2)
        .flatMap(m -> m.entrySet().stream())
        .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, Integer::sum));
```

**27. Find the top 3 highest-paid employees.**
```java
List<Employee> top3 = employees.stream()
        .sorted(Comparator.comparingDouble(Employee::salary).reversed())
        .limit(3)
        .toList();
```

**28. Compute min, max, and average salary in one pass.**
```java
DoubleSummaryStatistics stats = employees.stream()
        .mapToDouble(Employee::salary)
        .summaryStatistics();
System.out.printf("min=%.2f max=%.2f avg=%.2f%n", stats.getMin(), stats.getMax(), stats.getAverage());
```

**29. Find all pairs of numbers from two lists whose sum equals a target (nested `flatMap`).**
```java
List<int[]> pairs = list1.stream()
        .flatMap(a -> list2.stream()
                .filter(b -> a + b == target)
                .map(b -> new int[]{a, b}))
        .toList();
```

**30. Implement your own `Collectors.joining`-like custom Collector.**
```java
Collector<String, StringBuilder, String> joiner = Collector.of(
        StringBuilder::new,
        (sb, s) -> { if (sb.length() > 0) sb.append(", "); sb.append(s); },
        (sb1, sb2) -> { if (sb1.length() > 0 && sb2.length() > 0) sb1.append(", "); return sb1.append(sb2); }
);
String result = words.stream().collect(joiner);
```

---

## Bonus Pattern: Multi-Level Grouping + Downstream Transform
```java
// Department -> (Title -> average salary)
Map<String, Map<String, Double>> avgByDeptAndTitle = employees.stream()
        .collect(Collectors.groupingBy(Employee::department,
                Collectors.groupingBy(Employee::title,
                        Collectors.averagingDouble(Employee::salary))));
```

Next: [[22-Real-World-Examples]]
