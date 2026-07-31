---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Beginner
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Controllers & Request Mapping

## 💡 1-Sentence Elevator Pitch (For Interviews)
> `@RestController` combines `@Controller` and `@ResponseBody` to turn a Spring bean into an HTTP endpoint handler where each method is mapped to a URL pattern and HTTP verb via `@GetMapping`, `@PostMapping`, etc., with automatic JSON serialization via Jackson.

## 🧱 Key Concepts & Mechanics

### Core Annotations

| Annotation | Purpose |
|------------|---------|
| `@RestController` | `@Controller` + `@ResponseBody` — all methods return JSON/XML |
| `@RequestMapping` | Class or method level — sets base path / HTTP method |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@PatchMapping` / `@DeleteMapping` | Method-level shortcuts |
| `@PathVariable` | Binds URI segment `{id}` → method parameter |
| `@RequestParam` | Binds query string `?name=value` → parameter |
| `@RequestBody` | Deserializes request body JSON → object |
| `@RequestHeader` | Binds a specific HTTP header |
| `@ResponseStatus` | Sets default HTTP status code for a handler |

### Return Types
- Plain object → 200 OK + JSON body (Jackson serializes)
- `ResponseEntity<T>` → full control over status, headers, body
- `void` with `@ResponseStatus(HttpStatus.NO_CONTENT)` → 204

### Content Negotiation
- Spring MVC auto-negotiates based on `Accept` header and registered `HttpMessageConverter`s
- Jackson (`MappingJackson2HttpMessageConverter`) handles `application/json` by default

## 💻 Production Code Example

```java
@RestController
@RequestMapping("/api/v1/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    // Path variable + query params
    @GetMapping
    public List<ProductDto> list(
            @RequestParam(required = false) String category,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return productService.list(category, page, size);
    }

    @GetMapping("/{id}")
    public ProductDto getById(@PathVariable UUID id) {
        return productService.findById(id);  // throws 404 if missing
    }

    // Request body deserialization
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProductDto create(@Valid @RequestBody CreateProductRequest req) {
        return productService.create(req);
    }

    // Full ResponseEntity control
    @GetMapping("/{id}/export")
    public ResponseEntity<byte[]> exportAsPdf(@PathVariable UUID id) {
        byte[] pdf = productService.generatePdf(id);
        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"product-" + id + ".pdf\"")
            .contentType(MediaType.APPLICATION_PDF)
            .body(pdf);
    }

    // Custom request header
    @PostMapping("/bulk")
    public BulkResult bulkCreate(
            @RequestBody List<CreateProductRequest> reqs,
            @RequestHeader("X-Idempotency-Key") String idempotencyKey) {
        return productService.bulkCreate(reqs, idempotencyKey);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Putting business logic directly in the controller.
  **Fix:** Controllers are thin — delegate to `@Service` classes. Controllers handle HTTP concerns only (request parsing, response shaping).

- **Mistake:** Using `@RequestMapping` method shortcuts incorrectly — `@GetMapping` without a path defaults to the class-level path.
  **Fix:** Always be explicit about sub-paths; rely on the class-level `@RequestMapping` for the base only.

- **Mistake:** Returning `null` from a controller method — Jackson serializes it as an empty body with 200 OK.
  **Fix:** Return `ResponseEntity.notFound().build()` or throw a `ResponseStatusException(HttpStatus.NOT_FOUND)`.

- **Mistake:** Using `HttpServletRequest` / `HttpServletResponse` directly for common concerns.
  **Fix:** Use `@RequestHeader`, `@CookieValue`, `@RequestParam`. Reserve `HttpServletRequest` for unusual cases.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@Controller` and `@RestController`?
   - **A:** `@Controller` is used with view templates (Thymeleaf, JSP) — methods return view names. `@RestController` adds `@ResponseBody` to every method, so return values are serialized directly to the HTTP response body as JSON/XML.

2. **Q:** How does `@PathVariable` differ from `@RequestParam`?
   - **A:** `@PathVariable` extracts values from the URI path (`/users/{id}`). `@RequestParam` extracts values from the query string (`/users?id=123`). Path variables are typically used for resource identification; query params for filtering/pagination.

3. **Q:** How do you return a custom HTTP status from a controller method?
   - **A:** Either annotate the method with `@ResponseStatus(HttpStatus.CREATED)`, or return a `ResponseEntity<T>` with the desired status code.

## 🔗 Related Notes
- [[RESTful-API-Design]]
- [[Request-Validation-Jakarta]]
- [[Global-Exception-Handling]]
