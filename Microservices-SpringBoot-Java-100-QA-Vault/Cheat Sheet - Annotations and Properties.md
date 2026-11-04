---
tags: [microservices, springboot, cheatsheet]
---

# Cheat Sheet: Annotations and Properties

Quick lookup while coding. Back to [[00 - Index]].

## Spring core
| Annotation | Meaning |
|---|---|
| `@SpringBootApplication` | Main class: config + auto-config + component scan |
| `@Component` `@Service` `@Repository` | Create beans (by role) |
| `@Configuration` `@Bean` | Define beans manually |
| `@Autowired` | Inject (prefer constructor injection) |
| `@Value` `@ConfigurationProperties` | Read configuration |
| `@Profile` | Bean only in a profile |
| `@Scheduled` | Run a method on a timer |
| `@Async` | Run a method in another thread |

## Web
| Annotation | Meaning |
|---|---|
| `@RestController` | Controller returning JSON |
| `@GetMapping` `@PostMapping` `@PutMapping` `@PatchMapping` `@DeleteMapping` | Map HTTP methods |
| `@PathVariable` `@RequestParam` `@RequestBody` `@RequestHeader` | Read request data |
| `@Valid` `@NotBlank` `@NotNull` `@Positive` `@Email` | Validation |
| `@RestControllerAdvice` `@ExceptionHandler` | Global error handling |
| `@ResponseStatus` | Set status code |

## Data
| Annotation | Meaning |
|---|---|
| `@Entity` `@Id` `@GeneratedValue` | Map class to table |
| `@OneToMany` `@ManyToOne` | Relations |
| `@Transactional` | Transaction boundary |
| `@Version` | Optimistic locking |
| `@Query` `@EntityGraph` | Custom query / fetch plan |
| `@Cacheable` `@CacheEvict` | Caching |

## Cloud and messaging
| Annotation | Meaning |
|---|---|
| `@FeignClient` `@EnableFeignClients` | Declarative HTTP client |
| `@LoadBalanced` | Client-side load balancing |
| `@RefreshScope` | Reload config at runtime |
| `@CircuitBreaker` `@Retry` `@Bulkhead` `@RateLimiter` `@TimeLimiter` | Resilience4j |
| `@KafkaListener` | Consume Kafka messages |
| `@EnableMethodSecurity` `@PreAuthorize` | Method-level security |

## Testing
| Annotation | Meaning |
|---|---|
| `@SpringBootTest` | Full context test |
| `@WebMvcTest` `@DataJpaTest` | Test slices |
| `@MockitoBean` (Boot 3.4+) / `@MockBean` | Mock a Spring bean |
| `@Testcontainers` `@Container` `@ServiceConnection` | Real containers in tests |

## Useful properties
```yaml
server.port: 8080
server.shutdown: graceful
spring.application.name: order-service
spring.threads.virtual.enabled: true
spring.jpa.open-in-view: false
spring.jpa.hibernate.ddl-auto: validate
management.endpoints.web.exposure.include: health,info,prometheus
management.endpoint.health.probes.enabled: true
```
