---
tags: [microservices, springboot, java, interview, rest]
category: REST API Design
questions: 26-33
status: to-revise
---

# REST API Design (Q26 to Q33)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q26. What is REST and what makes a good REST API? 🟢
> REST is a style for building APIs over HTTP. Key ideas:
> - **Resources** are nouns in the URL: `GET /orders/5` (not `/getOrderById`).
> - Use standard HTTP methods and status codes.
> - **Stateless**: each request carries everything the server needs.
> - Use JSON, consistent names, and clear error messages.
>
> 📖 **Simple words:**
> - **Resource** = a thing your API manages, like an order or a customer.

> [!question]- Q27. Which HTTP methods are used and which are idempotent? 🟢
> | Method | Use | Idempotent? |
> |---|---|---|
> | GET | Read | Yes |
> | POST | Create | No |
> | PUT | Replace whole resource | Yes |
> | PATCH | Change part of resource | Usually no |
> | DELETE | Remove | Yes |
>
> Why it matters: if the network fails, a client can safely **retry** idempotent calls.
>
> 📖 **Simple words:**
> - **Idempotent** = doing it many times gives the same result as doing it once.

> [!question]- Q28. Which HTTP status codes should you know? 🟢
> | Code | Meaning |
> |---|---|
> | 200 OK | Success |
> | 201 Created | New resource created |
> | 204 No Content | Success, nothing to return |
> | 400 Bad Request | Invalid input |
> | 401 Unauthorized | Not logged in / bad token |
> | 403 Forbidden | Logged in, but not allowed |
> | 404 Not Found | Resource missing |
> | 409 Conflict | Duplicate or version conflict |
> | 429 Too Many Requests | Rate limit hit |
> | 500 Internal Server Error | Bug on server |
> | 503 Service Unavailable | Server overloaded or down |

> [!question]- Q29. What is the difference between @PathVariable, @RequestParam and @RequestBody? 🟢
> - `@PathVariable`: value inside the URL path: `/orders/5`
> - `@RequestParam`: value in the query string: `/orders?status=NEW`
> - `@RequestBody`: JSON sent in the request body (POST/PUT)
>
> ```java
> @GetMapping("/orders/{id}")
> OrderResponse get(@PathVariable Long id) { ... }
>
> @GetMapping("/orders")
> List<OrderResponse> search(@RequestParam String status) { ... }
>
> @PostMapping("/orders")
> OrderResponse create(@RequestBody CreateOrderRequest body) { ... }
> ```

> [!question]- Q30. How do you validate request data? 🟢
> Add `spring-boot-starter-validation`, put rules on the request class and add `@Valid` in the controller. If data is invalid, Spring throws `MethodArgumentNotValidException`, which you convert to a clean 400 response.
> Rule: **never trust input from outside**.
>
> 📖 **Simple words:**
> - **Validation** = checking that incoming data is correct before using it.
>
> ```java
> public record CreateOrderRequest(
>     @NotBlank String productId,
>     @Positive int quantity,
>     @Email String email) {}
>
> @PostMapping
> ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest req) { ... }
> ```

> [!question]- Q31. How do you handle exceptions globally? 🟡
> Use `@RestControllerAdvice` with `@ExceptionHandler` methods. One place converts exceptions into a standard error format. Spring 6 / Boot 3 supports **ProblemDetail** (RFC 7807), a standard JSON error shape. This keeps all your services consistent.
>
> 📖 **Simple words:**
> - **RFC 7807 (Problem Details)** = an official standard format for API error responses.
>
> ```java
> @RestControllerAdvice
> class GlobalExceptionHandler {
>     @ExceptionHandler(OrderNotFoundException.class)
>     ProblemDetail notFound(OrderNotFoundException ex) {
>         return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
>     }
>
>     @ExceptionHandler(MethodArgumentNotValidException.class)
>     ProblemDetail invalid(MethodArgumentNotValidException ex) {
>         ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation failed");
>         pd.setProperty("errors", ex.getBindingResult().getFieldErrors().stream()
>             .map(f -> f.getField() + ": " + f.getDefaultMessage()).toList());
>         return pd;
>     }
> }
> ```

> [!question]- Q32. Why use DTOs instead of exposing entities? 🟡
> An **entity** maps to a database table. If you return it directly:
> - You may leak internal or sensitive fields.
> - A DB change breaks your API.
> - Lazy-loading and infinite loops (A -> B -> A) can occur.
>
> A **DTO** is a separate object designed for the API. Use **MapStruct** to convert between them without writing boring code.
>
> 📖 **Simple words:**
> - **Entity** = Java class mapped to a database table.
> - **MapStruct** = a tool that generates mapping code between classes at compile time.

> [!question]- Q33. How do you version an API and add pagination? 🟡
> **Versioning:** put it in the URL (`/v1/orders`) or a header. Never break existing consumers; add new fields instead of removing old ones, and keep old versions until nobody uses them.
>
> **Pagination:** never return thousands of rows. Spring Data supports `Pageable` out of the box.
>
> 📖 **Simple words:**
> - **Consumer** = any client or service that calls your API.
> - **Pagination** = returning data in small pages.
>
> ```java
> @GetMapping("/v1/orders")
> Page<OrderResponse> list(@PageableDefault(size = 20, sort = "createdAt") Pageable pageable) {
>     return service.findAll(pageable);
> }
> // GET /v1/orders?page=0&size=20&sort=createdAt,desc
> ```

---
Previous: [[03 - Spring Boot Core]] | Next: [[05 - Data and Transactions]]
