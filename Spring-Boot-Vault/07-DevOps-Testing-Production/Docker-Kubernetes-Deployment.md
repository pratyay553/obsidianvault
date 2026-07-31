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

# 📌 Docker & Kubernetes Deployment

## 💡 1-Sentence Elevator Pitch (For Interviews)
> A multi-stage Dockerfile builds the Spring Boot fat JAR in a JDK image and copies only the runtime artifacts into a minimal JRE image — shrinking the production image by 60-80% — then Kubernetes orchestrates that image with rolling updates, health probes, resource limits, and ConfigMaps for environment-specific config.

## 🧱 Key Concepts & Mechanics

### Multi-Stage Dockerfile Strategy
1. **Build stage** — JDK + Maven/Gradle → produces the JAR
2. **Extract stage** — unpack the JAR into layered directories (Spring Boot 3 layered JARs)
3. **Runtime stage** — minimal JRE base (Distroless, Eclipse Temurin JRE) + layered JAR content

### Layered JAR (Spring Boot 3)
- Spring Boot automatically splits the JAR into layers: `dependencies`, `spring-boot-loader`, `snapshot-dependencies`, `application`
- Stable layers are cached by Docker — rebuilds only copy the `application` layer (your code)
- `java -Djarmode=tools -jar app.jar extract` — extracts layers

### Key Kubernetes Resources

| Resource | Purpose |
|----------|---------|
| `Deployment` | Manages ReplicaSet; rolling updates |
| `Service` (ClusterIP) | Stable internal DNS name for pods |
| `Ingress` | HTTP routing, TLS termination |
| `ConfigMap` | Non-sensitive config (env vars, config files) |
| `Secret` | Sensitive config (DB URL, API keys) |
| `HorizontalPodAutoscaler` | Scale replicas based on CPU/memory metrics |
| `PodDisruptionBudget` | Ensure min availability during node drain |

### Health Probes in K8s
- **Liveness**: Is the app alive? Failure → restart pod
- **Readiness**: Is the app ready for traffic? Failure → remove from load balancer
- **Startup**: Give slow apps time to start before liveness kicks in

### JVM Tuning for Containers
- Always set heap explicitly: `-Xms256m -Xmx512m`
- Or use `MaxRAMPercentage`: `-XX:MaxRAMPercentage=75.0` (recommended)
- G1GC is default in Java 17+; ZGC for low-latency (`-XX:+UseZGC`)
- Container-aware since Java 8u191: JVM reads cgroup limits automatically

## 💻 Production Code Example

```dockerfile
# ── Dockerfile (multi-stage + layered JAR) ────────────────────────────
# Stage 1: Build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /workspace

COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline -q   # cache dependencies

COPY src src
RUN ./mvnw package -DskipTests -q

# Stage 2: Extract layered JAR
FROM eclipse-temurin:21-jdk-alpine AS extractor
WORKDIR /workspace
COPY --from=builder /workspace/target/*.jar app.jar
RUN java -Djarmode=tools -jar app.jar extract --destination extracted

# Stage 3: Runtime (minimal — no JDK, no Maven, no build tools)
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

# Run as non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy layers in order of change frequency (stable → volatile)
COPY --from=extractor /workspace/extracted/dependencies/ ./
COPY --from=extractor /workspace/extracted/spring-boot-loader/ ./
COPY --from=extractor /workspace/extracted/snapshot-dependencies/ ./
COPY --from=extractor /workspace/extracted/application/ ./

EXPOSE 8080
ENTRYPOINT ["java", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseContainerSupport", \
  "-XX:+HeapDumpOnOutOfMemoryError", \
  "-XX:HeapDumpPath=/tmp/heapdump.hprof", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "org.springframework.boot.loader.launch.JarLauncher"]
```

```yaml
# ── Kubernetes Deployment ──────────────────────────────────────────────
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1         # allow 1 extra pod during update
      maxUnavailable: 0   # no downtime — always min replicas available
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: myregistry/order-service:1.2.3
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: order-service-secrets
                  key: database-url
            - name: SPRING_DATA_REDIS_HOST
              valueFrom:
                configMapKeyRef:
                  name: order-service-config
                  key: redis-host
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 60   # give JVM time to start
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 5
            failureThreshold: 3
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            failureThreshold: 30      # 30 * 10s = 5 minutes max startup time
            periodSeconds: 10
          volumeMounts:
            - name: config-volume
              mountPath: /app/config
      volumes:
        - name: config-volume
          configMap:
            name: order-service-config

---
# ── Service ──────────────────────────────────────────────────────────
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP

---
# ── HorizontalPodAutoscaler ───────────────────────────────────────────
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

```yaml
# ── Docker Compose (local dev) ────────────────────────────────────────
version: '3.9'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DATABASE_URL=jdbc:postgresql://postgres:5432/orders
      - SPRING_DATA_REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Running as root in the container — a container breakout gives full host access.
  **Fix:** Create a non-root user and switch: `USER spring:spring`.

- **Mistake:** No resource limits on pods — a memory leak in one pod starves the entire node.
  **Fix:** Always set `resources.requests` and `resources.limits`. Requests drive scheduling; limits prevent runaway usage.

- **Mistake:** Setting `-Xmx` without considering container limits — JVM can OOM-kill if `-Xmx` is set near the cgroup limit.
  **Fix:** Use `-XX:MaxRAMPercentage=75.0` — JVM auto-detects container memory and takes 75%.

- **Mistake:** No startup probe — liveness probe fires during slow JVM startup, killing the pod in a restart loop.
  **Fix:** Add `startupProbe` with enough `failureThreshold * periodSeconds` to cover worst-case startup.

- **Mistake:** Storing secrets in `ConfigMap` — ConfigMaps are not encrypted.
  **Fix:** Use `Secret` (base64-encoded, optionally encrypted at rest via KMS/Vault). Reference with `secretKeyRef`.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is a multi-stage Dockerfile and why use it?
   - **A:** A multi-stage build uses multiple `FROM` stages — an early stage builds the artifact using a full JDK/build toolchain, and the final stage copies only the runtime artifact into a minimal JRE base image. The result is a much smaller production image (no Maven, no JDK, no source code) — typically 60-80% smaller.

2. **Q:** What is the difference between liveness and readiness probes?
   - **A:** Liveness checks if the JVM is alive and not deadlocked — failure restarts the pod. Readiness checks if the app is ready to receive traffic (DB connected, warmup complete) — failure removes the pod from the load balancer without restarting it. A slow startup should fail readiness (not liveness) to avoid restart loops.

3. **Q:** How do you configure Spring Boot apps for Kubernetes without hardcoding config?
   - **A:** Mount `ConfigMap` values as environment variables or volume-mounted `application.yml` files, and `Secret` values as environment variables. Spring Boot reads env vars with relaxed binding — `SPRING_DATASOURCE_URL` maps to `spring.datasource.url`.

4. **Q:** What is `PodDisruptionBudget` and when would you use it?
   - **A:** A PDB guarantees a minimum number of available pods during voluntary disruptions (node drain, rolling update, cluster upgrade). For example, `minAvailable: 2` ensures at least 2 replicas are always running. Critical for zero-downtime during Kubernetes upgrades.

## 🔗 Related Notes
- [[Actuator-Metrics-Prometheus-Grafana]]
- [[Unit-And-Integration-Testing-Testcontainers]]
- [[Configuration-And-Properties]]
