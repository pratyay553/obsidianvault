---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Intermediate
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 RESTful API Design

## 💡 1-Sentence Elevator Pitch (For Interviews)
> REST is an architectural style where resources are identified by URIs, manipulated via standard HTTP verbs, and represented in self-descriptive formats — Spring Boot's `@RestController` maps this directly to Java methods.

## 🧱 Key Concepts & Mechanics

### HTTP Verbs & Semantics

| Verb | Idempotent | Safe | Use |
|------|-----------|------|-----|
| `GET` | ✅ | ✅ | Read resource |
| `POST` | ❌ | ❌ | Create resource |
| `PUT` | ✅ | ❌ | Full replace |
| `PATCH` | ❌ | ❌ | Partial update |
| `DELETE` | ✅ | ❌ | Remove resource |

### Resource URI Conventions
- Nouns, not verbs: `/orders` not `/getOrders`
- Plural collections: `/users`, `/products`
- Nested sub-resources: `/users/{userId}/orders/{orderId}`
- Actions that don't fit CRUD: `/orders/{id}/cancel` (POST)
- Filtering via query params: `GET /products?category=electronics&sort=price`

### HTTP Status Codes

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | GET/PUT/PATCH success |
| 201 | Created | POST success — include `Location` header |
| 204 | No Content | DELETE success |
| 400 | Bad Request | Invalid input / validation failure |
| 401 | Unauthorized | Missing / invalid auth token |
| 403 | Forbidden | Authenticated but lacks permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate / state conflict |
| 422 | Unprocessable Entity | Semantic validation failure |
| 500 | Internal Server Error | Unexpected server error |

### API Versioning Strategies
- **URI versioning:** `/api/v1/users` — most common, easy to route
- **Header versioning:** `Accept: application/vnd.myapp.v1+json`
- **Query param:** `/users?version=1` — avoid, pollutes resource URIs

## 💻 Production Code Example

```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    // GET /api/v1/orders?status=PENDING&page=0&size=20
    @GetMapping
    public Page<OrderSummaryDto> list(
            @RequestParam(defaultValue = "ALL") OrderStatus status,
            Pageable pageable) {
        return orderService.findAll(status, pageable);
    }

    // GET /api/v1/orders/{id}
    @GetMapping("/{id}")
    public OrderDetailDto get(@PathVariable UUID id) {
        return orderService.findById(id);
    }

    // POST /api/v1/orders  → 201 Created + Location header
    @PostMapping
    public ResponseEntity<OrderDetailDto> create(
            @Valid @RequestBody CreateOrderRequest req) {
        OrderDetailDto order = orderService.create(req);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(order.id()).toUri();
        return ResponseEntity.created(location).body(order);
    }

    // PATCH /api/v1/orders/{id}
    @PatchMapping("/{id}")
    public OrderDetailDto patch(
            @PathVariable UUID id,
            @Valid @RequestBody PatchOrderRequest req) {
        return orderService.patch(id, req);
    }

    // POST /api/v1/orders/{id}/cancel  (action endpoint)
    @PostMapping("/{id}/cancel")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void cancel(@PathVariable UUID id) {
        orderService.cancel(id);
    }

    // DELETE /api/v1/orders/{id}  → 204 No Content
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable UUID id) {
        orderService.delete(id);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using verbs in URIs (`/getUser`, `/createOrder`).
  **Fix:** URIs identify resources; verbs come from HTTP methods.

- **Mistake:** Returning `200 OK` with `{ "success": false, "error": "..." }` bodies.
  **Fix:** Use proper HTTP status codes — the HTTP spec already encodes success/failure.

- **Mistake:** Exposing internal entity IDs (auto-increment longs) in public APIs.
  **Fix:** Use UUIDs or opaque string IDs to prevent enumeration attacks.

- **Mistake:** Forgetting the `Location` header on `201 Created` responses.
  **Fix:** Always include `Location: /api/v1/resource/{id}` so clients know where to find the new resource.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between PUT and PATCH?
   - **A:** PUT replaces the entire resource — any field not included is set to null/default. PATCH applies a partial update — only the fields included in the request body are changed.

2. **Q:** What does idempotent mean and why does it matter?
   - **A:** An idempotent operation produces the same result regardless of how many times it's called. GET, PUT, DELETE are idempotent — retrying on network failure is safe. POST is not — retrying could create duplicates.

3. **Q:** When would you use 422 vs 400?
   - **A:** 400 for syntactically malformed requests (unparseable JSON, missing required field). 422 for requests that are syntactically valid but semantically wrong (e.g., transferring to an account that doesn't accept transfers).

## 🔗 Related Notes
- [[Controllers-And-Request-Mapping]]
- [[Request-Validation-Jakarta]]
- [[Global-Exception-Handling]]
