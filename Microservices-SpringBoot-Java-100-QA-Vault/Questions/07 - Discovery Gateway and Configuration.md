---
tags: [microservices, springboot, java, interview, infrastructure]
category: Discovery, Gateway and Configuration
questions: 51-58
status: to-revise
---

# Discovery, Gateway and Configuration (Q51 to Q58)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q51. What is service discovery and how does Eureka work? 🟢
> In the cloud, service instances start, stop and change IP addresses all the time. **Service discovery** is a phone book: each service **registers** its name and address, and others **look it up** by name (`inventory-service`).
> **Eureka** (Netflix/Spring Cloud) is a registry server. On Kubernetes you usually do not need it, because Kubernetes **Services + DNS** already do this.
>
> 📖 **Simple words:**
> - **Registry** = a list of running services and where they are.
> - **DNS** = the internet's phone book that turns names into addresses.
>
> ```yaml
> # client
> eureka:
>   client:
>     service-url:
>       defaultZone: http://eureka:8761/eureka
> ```

> [!question]- Q52. What is load balancing? Client-side vs server-side? 🟡
> **Load balancing** spreads requests over many instances of a service.
> - **Server-side:** a separate load balancer (Nginx, cloud LB, Kubernetes Service) sits in the middle.
> - **Client-side:** the calling service gets the list of instances and picks one itself (**Spring Cloud LoadBalancer**).
>
> ```java
> @Bean @LoadBalanced
> RestClient.Builder restClientBuilder() { return RestClient.builder(); }
>
> // "inventory-service" is resolved to a real instance
> restClient.get().uri("http://inventory-service/stock/{id}", id).retrieve().body(Stock.class);
> ```

> [!question]- Q53. What is Spring Cloud Gateway? 🟢
> A single **entry point** for all clients. It routes requests to the right service and handles shared jobs: authentication, rate limiting, logging, CORS, header changes.
> Main terms: **Route** (where to send), **Predicate** (when to match, like a path), **Filter** (change request/response).
> Keep business logic out of the gateway.
>
> ```yaml
> spring:
>   cloud:
>     gateway:
>       routes:
>         - id: orders
>           uri: lb://order-service
>           predicates:
>             - Path=/api/orders/**
>           filters:
>             - StripPrefix=1
> ```

> [!question]- Q54. API Gateway vs Load Balancer vs Reverse Proxy: what is the difference? 🟡
> - **Reverse proxy:** receives requests for servers and forwards them (Nginx).
> - **Load balancer:** a reverse proxy that spreads traffic across **copies of the same service**.
> - **API Gateway:** adds API-level features: routing to **different** services, auth, rate limiting, request transformation.
>
> In practice they are layered: Load balancer -> API Gateway -> services.

> [!question]- Q55. What is rate limiting and why is it needed? 🟡
> Rate limiting restricts how many requests a client can send in a time window (for example 20 per second). It protects your system from abuse, bugs in clients and traffic spikes. Over the limit, return **429 Too Many Requests**.
> A common algorithm is the **token bucket**: tokens refill at a fixed speed; each request uses one.
>
> 📖 **Simple words:**
> - **Throttling** = slowing or rejecting requests to protect the system.
>
> ```yaml
> filters:
>   - name: RequestRateLimiter
>     args:
>       redis-rate-limiter.replenishRate: 20
>       redis-rate-limiter.burstCapacity: 40
>       key-resolver: "#{@userKeyResolver}"
> ```

> [!question]- Q56. How do you manage configuration centrally (Config Server, ConfigMap, refresh)? 🟡
> - **Spring Cloud Config Server** reads settings from a Git repo and serves them to all services.
> - On Kubernetes, use **ConfigMaps** (normal settings) and **Secrets** (passwords) instead.
> - `@RefreshScope` beans can reload changed values without a restart (after calling `/actuator/refresh`).
>
> ```yaml
> spring:
>   config:
>     import: "optional:configserver:http://config-server:8888"
> ```
> ```java
> @RefreshScope
> @RestController
> class FeatureController {
>     @Value("${feature.new-checkout:false}") boolean enabled;
> }
> ```

> [!question]- Q57. How should you handle secrets like passwords and API keys? 🟡
> - Never put secrets in Git or in the JAR.
> - Inject them at runtime through environment variables, Kubernetes Secrets, or a vault tool such as **HashiCorp Vault** or a cloud secret manager.
> - Rotate them regularly, give each service its own credentials, and never log them.
>
> 📖 **Simple words:**
> - **Secret** = sensitive value (password, token, key).
> - **Rotate** = change a secret regularly.

> [!question]- Q58. What is a service mesh (Istio) and do you need one? 🔴
> A service mesh moves network features (mTLS encryption, retries, timeouts, traffic splitting, metrics) out of your code into a **sidecar proxy** that runs next to each service. Examples: Istio, Linkerd.
> You need it when you have **many** services and want uniform security and traffic control. For a few services, Spring Cloud and Kubernetes features are enough, because a mesh adds operational complexity.
>
> 📖 **Simple words:**
> - **Sidecar** = a helper container running beside your app container.
> - **mTLS** = both sides prove their identity and the traffic is encrypted.

---
Previous: [[06 - Service Communication and Messaging]] | Next: [[08 - Resilience and Distributed Data]]
