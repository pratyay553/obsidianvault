---
tags: [microservices, springboot, java, interview, devops]
category: Docker, Kubernetes and DevOps
questions: 88-93
status: to-revise
---

# Docker, Kubernetes and DevOps (Q88 to Q93)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q88. How do you containerize a Spring Boot application? 🟢
> A **container** packages your app with everything it needs to run, so it behaves the same everywhere. Two ways:
> 1. Write a **Dockerfile**.
> 2. Use **Cloud Native Buildpacks**: `./mvnw spring-boot:build-image` (no Dockerfile, good defaults).
>
> Good habits: small base image, run as non-root user, set JVM memory flags.
>
> 📖 **Simple words:**
> - **Image** = a packaged snapshot of your app; a **container** is a running image.
>
> ```dockerfile
> FROM eclipse-temurin:21-jre
> WORKDIR /app
> COPY target/order-service.jar app.jar
> USER 1000
> ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75", "-jar", "app.jar"]
> ```

> [!question]- Q89. What are the basic Kubernetes objects every developer should know? 🟡
> | Object | Meaning |
> |---|---|
> | **Pod** | Smallest unit; runs one or more containers |
> | **Deployment** | Keeps N copies of a Pod running; handles rolling updates |
> | **Service** | Stable name/IP and load balancing for Pods |
> | **Ingress** | Routes outside traffic into the cluster |
> | **ConfigMap / Secret** | Settings / sensitive settings |
> | **HPA** | Autoscaler: adds/removes Pods by CPU or custom metric |
> | **Namespace** | Logical separation (dev, prod) |
>
> 📖 **Simple words:**
> - **Kubernetes (K8s)** = a system that runs and manages containers at scale.

> [!question]- Q90. What are liveness and readiness probes? Why graceful shutdown? 🟡
> - **Liveness probe:** "Is the app alive?" If it fails, Kubernetes **restarts** the Pod.
> - **Readiness probe:** "Can the app take traffic now?" If it fails, the Pod is **removed from the load balancer** (not restarted). Use it during startup or when a dependency is down.
>
> **Graceful shutdown** lets running requests finish before the Pod stops, so deployments cause no errors.
>
> ```yaml
> # application.yml
> server.shutdown: graceful
> spring.lifecycle.timeout-per-shutdown-phase: 30s
> management.endpoint.health.probes.enabled: true
> ```
> ```yaml
> # Kubernetes
> livenessProbe:  { httpGet: { path: /actuator/health/liveness,  port: 8080 } }
> readinessProbe: { httpGet: { path: /actuator/health/readiness, port: 8080 } }
> ```

> [!question]- Q91. What does a CI/CD pipeline for a microservice look like? 🟡
> **CI (Continuous Integration):** every commit is built and tested automatically. **CD (Continuous Delivery/Deployment):** the tested build is released automatically.
> Typical stages for **each service separately**:
> 1. Build and unit tests
> 2. Integration and contract tests
> 3. Code quality and security scan (SonarQube, dependency and image scan)
> 4. Build and push Docker image
> 5. Deploy to test, then production (GitOps with Argo CD / Flux)
>
> Tools: GitHub Actions, GitLab CI, Jenkins.
>
> 📖 **Simple words:**
> - **GitOps** = the desired state of the system is stored in Git; a tool applies it.

> [!question]- Q92. What are rolling, blue-green and canary deployments? 🟡
> - **Rolling:** replace old Pods with new ones a few at a time. Default in Kubernetes.
> - **Blue-green:** run old (blue) and new (green) side by side, then switch all traffic at once. Instant rollback.
> - **Canary:** send a small part of traffic (say 5%) to the new version first. If metrics look good, increase.
>
> Add **feature flags** to release code without turning the feature on yet.

> [!question]- Q93. How do you scale microservices? 🟡
> - **Horizontal scaling:** add more instances (Pods). Preferred; needs **stateless** services.
> - **Vertical scaling:** give one instance more CPU/RAM. Has limits.
> - Use **HPA** to scale automatically by CPU or custom metrics (like request rate or Kafka lag).
> - Set resource **requests and limits**. Scale the **database** carefully too; it is often the real bottleneck (use read replicas, caching, partitioning).
>
> 📖 **Simple words:**
> - **Read replica** = a copy of a database used only for reading.
>
> ```yaml
> resources:
>   requests: { cpu: 250m, memory: 512Mi }
>   limits:   { memory: 768Mi }
> ```

---
Previous: [[11 - Observability and Monitoring]] | Next: [[13 - RealWorld Scenarios]]
