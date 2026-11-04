---
tags: [microservices, springboot, java, interview, testing]
category: Testing
questions: 77-82
status: to-revise
---

# Testing (Q77 to Q82)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q77. What is the test pyramid? 🟢
> A guide for how many tests of each type to write:
> - **Many unit tests** (fast, test one class, no Spring).
> - **Some integration tests** (service + real DB/broker, using Testcontainers).
> - **Few end-to-end tests** (full system, slow, fragile).
>
> Fast tests give quick feedback; slow tests should protect only the most important flows.

> [!question]- Q78. @SpringBootTest vs @WebMvcTest vs @DataJpaTest: what is the difference? 🟡
> | Annotation | Loads | Use for |
> |---|---|---|
> | `@SpringBootTest` | Whole app | Full integration tests (slow) |
> | `@WebMvcTest` | Only web layer (controllers) | Controller tests with MockMvc |
> | `@DataJpaTest` | Only JPA + in-memory/test DB | Repository tests |
> | `@JsonTest` | Only JSON mapping | Serialization tests |
>
> These are called **test slices**: load only what you need to keep tests fast.

> [!question]- Q79. How do you use Mockito to unit test a service? 🟢
> **Mockito** creates fake objects (**mocks**) for dependencies, so you test only your own class, with no database or network.
>
> 📖 **Simple words:**
> - **Mock** = a fake object that you control in a test.
>
> ```java
> @ExtendWith(MockitoExtension.class)
> class OrderServiceTest {
>     @Mock PaymentClient paymentClient;
>     @InjectMocks OrderService service;
>
>     @Test
>     void failsWhenPaymentDeclined() {
>         when(paymentClient.pay(any())).thenThrow(new PaymentDeclinedException());
>         assertThrows(PaymentDeclinedException.class, () -> service.place(request));
>     }
> }
> ```

> [!question]- Q80. What is Testcontainers and why use it? 🟡
> Testcontainers starts **real Docker containers** (PostgreSQL, Kafka, Redis) during tests. It is far more trustworthy than an in-memory H2 database, because H2 behaves differently from production databases. Spring Boot 3.1+ connects automatically with `@ServiceConnection`.
>
> ```java
> @SpringBootTest
> @Testcontainers
> class OrderRepositoryIT {
>     @Container @ServiceConnection
>     static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
>
>     @Autowired OrderRepository repository;
>
>     @Test void savesOrder() {
>         assertThat(repository.save(new OrderEntity()).getId()).isNotNull();
>     }
> }
> ```

> [!question]- Q81. What is contract testing? 🔴
> With many services, how do you know a change will not break a consumer? **Contract testing** checks that provider and consumer agree on the API. In **consumer-driven contracts**, the consumer writes what it expects; the provider's build runs those checks. Tools: **Pact**, **Spring Cloud Contract**. It catches breaking changes early without running every service together.
>
> 📖 **Simple words:**
> - **Provider** = the service that offers an API.
> - **Contract** = the agreed request/response format.

> [!question]- Q82. How do you test a REST controller with MockMvc? 🟢
> Use `@WebMvcTest` to load only the controller and mock the service. Then call the endpoint with `MockMvc`.
>
> ```java
> @WebMvcTest(OrderController.class)
> class OrderControllerTest {
>     @Autowired MockMvc mvc;
>     @MockitoBean OrderService service;   // Boot 3.4+. In older versions use @MockBean
>
>     @Test
>     void returns404WhenMissing() throws Exception {
>         when(service.get(1L)).thenThrow(new OrderNotFoundException(1L));
>         mvc.perform(get("/orders/1")).andExpect(status().isNotFound());
>     }
> }
> ```

---
Previous: [[09 - Security]] | Next: [[11 - Observability and Monitoring]]
