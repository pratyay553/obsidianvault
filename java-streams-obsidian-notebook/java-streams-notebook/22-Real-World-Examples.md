---
tags: [java, streams, real-world]
---

# 22. Real-World Examples

⬅ Back to [[00-Home]] | Prev: [[21-Coding-Problems]] | Next: [[23-Cheat-Sheet]]

## 1. REST API — Filtering and Shaping a Response DTO

```java
public List<UserSummaryDto> getActiveUserSummaries(List<User> users) {
    return users.stream()
            .filter(User::isActive)
            .sorted(Comparator.comparing(User::getLastLoginAt).reversed())
            .map(u -> new UserSummaryDto(u.getId(), u.getName(), u.getEmail()))
            .toList();
}
```

## 2. Log File Analysis — Counting Error Levels

```java
try (Stream<String> lines = Files.lines(Path.of("app.log"))) {
    Map<String, Long> countByLevel = lines
            .filter(line -> line.contains("ERROR") || line.contains("WARN"))
            .map(line -> line.split("\\s+")[1]) // e.g., "ERROR" token
            .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
    countByLevel.forEach((level, count) -> System.out.println(level + ": " + count));
}
```

## 3. E-commerce — Order Aggregation & Revenue Report

```java
public Map<String, Double> revenueByRegion(List<Order> orders) {
    return orders.parallelStream()
            .filter(o -> o.getStatus() == OrderStatus.COMPLETED)
            .collect(Collectors.groupingBy(Order::getRegion,
                    Collectors.summingDouble(Order::getAmount)));
}
```

## 4. Batch ETL — Transform CSV Rows into Domain Objects

```java
public List<Product> parseCsv(List<String> csvLines) {
    return csvLines.stream()
            .skip(1) // header
            .filter(line -> !line.isBlank())
            .map(line -> line.split(","))
            .map(fields -> new Product(fields[0], Double.parseDouble(fields[1]), Integer.parseInt(fields[2])))
            .toList();
}
```

## 5. Inventory Management — Low Stock Alerts

```java
public List<String> lowStockAlerts(List<Product> products, int threshold) {
    return products.stream()
            .filter(p -> p.getQuantity() < threshold)
            .map(p -> "LOW STOCK: " + p.getName() + " (" + p.getQuantity() + " left)")
            .toList();
}
```

## 6. Analytics Dashboard — Multi-Level Grouping

```java
// Region -> Category -> total sales
Map<String, Map<String, Double>> salesByRegionAndCategory = sales.stream()
        .collect(Collectors.groupingBy(Sale::getRegion,
                Collectors.groupingBy(Sale::getCategory,
                        Collectors.summingDouble(Sale::getAmount))));
```

## 7. Deduplication with Business-Key Semantics

```java
// Keep only the most recent record per customer ID
Map<String, Customer> latestByCustomerId = customers.stream()
        .collect(Collectors.toMap(Customer::getId, c -> c,
                (existing, incoming) -> incoming.getUpdatedAt().isAfter(existing.getUpdatedAt()) ? incoming : existing));
```

## 8. Pagination Utility

```java
public <T> List<T> paginate(List<T> source, int page, int size) {
    return source.stream()
            .skip((long) page * size)
            .limit(size)
            .toList();
}
```

## 9. Validation Pipeline — Collecting All Errors (not just the first)

```java
public List<String> validate(Order order) {
    return Stream.<Function<Order, Optional<String>>>of(
                o -> o.getAmount() <= 0 ? Optional.of("Amount must be positive") : Optional.empty(),
                o -> o.getItems().isEmpty() ? Optional.of("Order must have items") : Optional.empty(),
                o -> o.getCustomerId() == null ? Optional.of("Customer ID required") : Optional.empty()
            )
            .map(validator -> validator.apply(order))
            .flatMap(Optional::stream)
            .toList();
}
```

## 10. Parallel Image/Data Processing Pipeline (CPU-bound, large batch)

```java
public List<ProcessedImage> processImagesInParallel(List<RawImage> images) {
    return images.parallelStream()
            .map(this::resize)          // CPU-bound, independent per element
            .map(this::applyFilters)
            .map(this::compress)
            .toList();
}
```
See [[11-Parallel-Streams]] for when this pattern is appropriate (large N, CPU-bound, stateless per-element work).

## 11. Building a Lookup Index from a List

```java
Map<String, List<Order>> ordersByCustomer = orders.stream()
        .collect(Collectors.groupingBy(Order::getCustomerId));
```

## 12. Combining Streams from Multiple Sources

```java
Stream<String> combined = Stream.concat(listA.stream(), listB.stream())
        .distinct()
        .sorted();
```

## 13. Statistics Report for a Dashboard

```java
DoubleSummaryStatistics orderStats = orders.stream()
        .mapToDouble(Order::getAmount)
        .summaryStatistics();

Report report = new Report(orderStats.getCount(), orderStats.getSum(),
                            orderStats.getAverage(), orderStats.getMin(), orderStats.getMax());
```

Next: [[23-Cheat-Sheet]]
