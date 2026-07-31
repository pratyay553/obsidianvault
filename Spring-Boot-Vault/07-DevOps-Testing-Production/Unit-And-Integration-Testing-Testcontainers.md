---
tags:
  - springboot
  - java
  - backend
  - interview-prep
technology: Spring Boot 3.x
level: Advanced
status: Not Started
last_reviewed: 2026-07-23
---

# 📌 Unit & Integration Testing with Testcontainers

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Boot testing uses `@SpringBootTest` for full-context integration tests, `@WebMvcTest` for slice tests (controllers only), and Testcontainers to spin up real Docker containers for PostgreSQL and Redis — so tests run against the same database engine as production, not an in-memory H2 approximation.

## 🧱 Key Concepts & Mechanics

### Test Slice Annotations (load partial context)

| Annotation | What it loads | Use For |
|------------|--------------|---------|
| `@SpringBootTest` | Full `ApplicationContext` | End-to-end integration tests |
| `@WebMvcTest(Controller.class)` | MVC layer only (controller, filters, security) | Controller unit tests |
| `@DataJpaTest` | JPA layer only (repos, entities) | Repository tests |
| `@JsonTest` | Jackson serialization | DTO/JSON mapping tests |
| `@MockBean` | Spring bean mock (replaces real bean in context) | Isolate dependencies |
| `@SpyBean` | Partial mock — real bean with selective override | Verify interactions |

### MockMvc vs WebTestClient vs RestAssured
- `MockMvc` — synchronous, no real HTTP server, fast; pairs with `@WebMvcTest`
- `WebTestClient` — reactive/non-blocking, works with WebFlux; also usable with `@SpringBootTest`
- `RestAssured` — readable DSL, real HTTP; requires `webEnvironment = RANDOM_PORT`

### Testcontainers
- Launches real Docker containers in JUnit 5 lifecycle
- `@Container` (static) — shared across all tests in class (faster)
- `@Container` (instance) — new container per test (isolated but slow)
- `DynamicPropertySource` — injects container port into Spring properties
- Spring Boot 3.1+: `@ServiceConnection` auto-wires connection without `DynamicPropertySource`

### Test Layering Strategy
```
Unit Tests (services, domain logic)   → Mock all dependencies → JUnit 5 + Mockito
Controller Tests (@WebMvcTest)        → Mock service layer   → MockMvc + @MockBean
Repository Tests (@DataJpaTest)       → Real DB              → Testcontainers PostgreSQL
Integration Tests (@SpringBootTest)   → Full stack           → Testcontainers all deps
```

## 💻 Production Code Example

```java
// ── Unit test — pure Java, no Spring ─────────────────────────────────
class OrderServiceTest {

    @Mock OrderRepository orderRepo;
    @Mock InventoryService inventoryService;
    @InjectMocks OrderService orderService;

    @BeforeEach
    void setUp() { MockitoAnnotations.openMocks(this); }

    @Test
    void placeOrder_shouldReserveInventoryAndSave() {
        PlaceOrderRequest req = new PlaceOrderRequest(List.of(new Item("prod-1", 2)));
        when(inventoryService.reserve(any())).thenReturn(InventoryResult.success());
        when(orderRepo.save(any())).thenAnswer(inv -> inv.getArgument(0));

        OrderDto result = orderService.placeOrder(req);

        verify(inventoryService).reserve(req.items());
        verify(orderRepo).save(any(Order.class));
        assertThat(result.status()).isEqualTo(OrderStatus.PENDING);
    }

    @Test
    void placeOrder_shouldThrowWhenInventoryFails() {
        when(inventoryService.reserve(any()))
            .thenThrow(new InsufficientStockException("prod-1"));
        assertThatThrownBy(() -> orderService.placeOrder(anyRequest()))
            .isInstanceOf(InsufficientStockException.class);
        verify(orderRepo, never()).save(any());
    }
}

// ── Controller test (@WebMvcTest) ─────────────────────────────────────
@WebMvcTest(OrderController.class)
@Import(SecurityConfig.class)
class OrderControllerTest {

    @Autowired MockMvc mockMvc;
    @Autowired ObjectMapper objectMapper;
    @MockBean OrderService orderService;

    @Test
    @WithMockUser
    void createOrder_shouldReturn201WithLocation() throws Exception {
        UUID id = UUID.randomUUID();
        when(orderService.placeOrder(any())).thenReturn(OrderDto.builder().id(id).build());

        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(validRequest())))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(id.toString()));
    }

    @Test
    @WithMockUser
    void createOrder_shouldReturn400OnInvalidBody() throws Exception {
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))  // missing required fields
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").exists());
    }
}

// ── Repository test with Testcontainers ──────────────────────────────
@DataJpaTest
@AutoConfigureTestDatabase(replace = NONE)  // don't replace with H2
class OrderRepositoryTest {

    @Container
    @ServiceConnection   // Spring Boot 3.1+ auto-configures datasource
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired OrderRepository orderRepo;
    @Autowired TestEntityManager em;

    @Test
    void findByStatus_shouldReturnMatchingOrders() {
        Order order = createOrder(OrderStatus.PENDING);
        em.persistAndFlush(order);

        List<Order> result = orderRepo.findByStatus(OrderStatus.PENDING,
            PageRequest.of(0, 10)).getContent();

        assertThat(result).hasSize(1);
        assertThat(result.get(0).getId()).isEqualTo(order.getId());
    }
}

// ── Full integration test ─────────────────────────────────────────────
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    @ServiceConnection
    static RedisContainer redis = new RedisContainer("redis:7-alpine");

    @Autowired TestRestTemplate restTemplate;

    @Test
    void fullOrderFlow_shouldCreateAndRetrieveOrder() {
        // 1. Create order
        ResponseEntity<OrderDto> created = restTemplate.withBasicAuth("user", "pass")
            .postForEntity("/api/v1/orders", validRequest(), OrderDto.class);

        assertThat(created.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        UUID orderId = created.getBody().id();

        // 2. Retrieve order
        ResponseEntity<OrderDto> retrieved = restTemplate.withBasicAuth("user", "pass")
            .getForEntity("/api/v1/orders/" + orderId, OrderDto.class);

        assertThat(retrieved.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(retrieved.getBody().id()).isEqualTo(orderId);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using H2 in-memory database for JPA tests — H2 and PostgreSQL have different SQL dialects (e.g., `SERIAL`, `JSONB`, generated columns don't work in H2).
  **Fix:** Use Testcontainers PostgreSQL with `@AutoConfigureTestDatabase(replace = NONE)`.

- **Mistake:** `@SpringBootTest` for every test — loads the entire context for each test class, making the suite slow.
  **Fix:** Use `@WebMvcTest` for controllers, `@DataJpaTest` for repos, pure JUnit+Mockito for services.

- **Mistake:** `@Container` as an instance field — starts a new container per test method.
  **Fix:** Make containers `static` to share across the test class. Use `@Testcontainers` + `@Container` (static).

- **Mistake:** `@MockBean` invalidates the Spring context cache — each unique `@MockBean` combination creates a new context.
  **Fix:** Centralize all `@MockBean` declarations in a shared base test class to maximize context reuse.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is the difference between `@Mock` (Mockito) and `@MockBean` (Spring)?
   - **A:** `@Mock` (Mockito) creates a mock outside of Spring's context — fast, pure unit test. `@MockBean` creates a mock and registers it in the Spring `ApplicationContext`, replacing the real bean — needed when testing with `@WebMvcTest` or `@SpringBootTest` where Spring manages the wiring.

2. **Q:** Why use Testcontainers instead of H2?
   - **A:** H2 is not PostgreSQL — it doesn't support native types (`JSONB`, `UUID`, `SERIAL`), some SQL features, or PostgreSQL-specific behavior (constraint handling, isolation levels). Testcontainers runs the actual PostgreSQL image, giving you a faithful test environment that matches production.

3. **Q:** What is `@DataJpaTest` and what does it load?
   - **A:** A Spring Boot test slice that loads only JPA repositories, entities, the `EntityManager`, and DataSource — not controllers, services, or security. It applies `@AutoConfigureTestDatabase` (replaces with H2 by default — override with `replace = NONE` + Testcontainers). Ideal for testing repository methods in isolation.

4. **Q:** What is context caching in Spring tests and why does `@MockBean` break it?
   - **A:** Spring caches the `ApplicationContext` across test classes with the same configuration to avoid reloading. `@MockBean` with a different mock type creates a distinct context configuration, forcing a new context load. Group tests by mock requirements or use a shared base class.

## 🔗 Related Notes
- [[Spring-Data-JPA-Fundamentals]]
- [[Spring-Security-Architecture-Filters]]
- [[Docker-Kubernetes-Deployment]]
