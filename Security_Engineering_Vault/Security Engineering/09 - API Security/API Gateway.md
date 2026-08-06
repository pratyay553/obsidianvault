# API Gateway

A simple guide to API Gateways, written for easy understanding.

---

## Definition

An API Gateway is a server that sits in front of your backend services. Every request from a client goes through the gateway first. The gateway decides what to do with each request before it reaches the real service.

Think of it like a receptionist at an office. Visitors do not walk straight to any employee. They first talk to the receptionist, who checks their identity, decides which department to send them to, and keeps a record of every visit.

---

## Why it Matters

- **One entry point** — All requests go through one place. This makes the whole system easier to manage and secure.
- **Less duplicate code** — Without a gateway, every microservice would need to write its own login checks, rate limits, and logging. The gateway does this once, for everyone.
- **Easier to change things** — You can swap out a backend service, change its address, or update its version, without the client ever knowing.
- **Better monitoring** — Since all traffic passes through one point, it is easy to collect logs, metrics, and alerts in one place.
- **Stronger security** — Bad or unauthorized requests can be stopped at the gateway, before they even reach your real business logic.

---

## How it Works

A typical request flow looks like this:

1. **Client sends a request** to the gateway's public address, not directly to the backend service.
2. **Authentication check** — The gateway checks if the request has a valid token or API key.
3. **Authorization check** — The gateway checks if this user or app is allowed to do this specific action.
4. **Rate limiting** — The gateway checks if this client has sent too many requests recently.
5. **Routing** — The gateway decides which backend service should handle this request, based on the URL path or other rules.
6. **Request transformation** (optional) — The gateway may change the request format, add headers, or convert protocols before sending it onward.
7. **Backend service responds** — The real service processes the request and sends back a response.
8. **Response transformation** (optional) — The gateway may reshape the response before sending it back to the client.
9. **Logging** — The gateway records details about the request and response for monitoring.

Common gateway tools include **Spring Cloud Gateway**, **Kong**, **NGINX**, **AWS API Gateway**, and **Apigee**.

---

## Common Pitfalls

- **Treating the gateway as the only security layer** — Backend services should still check permissions themselves. If the gateway is ever bypassed or misconfigured, the backend must not be left wide open.
- **Single point of failure** — If the gateway goes down, every service behind it becomes unreachable. It needs to be highly available, usually with multiple instances behind a load balancer.
- **Slow performance** — Adding too much logic inside the gateway (heavy transformations, slow external calls) can make every single request slower.
- **Bad CORS settings** — Allowing any website to call your API through the gateway can open the door to attacks from malicious sites.
- **Forgetting to remove old routes** — Old or unused API versions left running behind the gateway become a hidden security risk, since nobody is watching them anymore.
- **Not validating input at the gateway** — Some teams assume the backend will handle bad input, but catching bad data early, at the gateway, is safer and cheaper.
- **Hardcoding backend addresses** — Makes it hard to move, scale, or replace services later. Use service discovery or configuration instead.

---

## Spring Boot / Java Notes

Spring provides a project called **Spring Cloud Gateway**, built specifically for this purpose.

- It is built on **Spring WebFlux**, which is non-blocking. This means it can handle many requests at the same time, without wasting threads.
- **Routes** are defined using `RouteLocator`, either in Java code or in the `application.yml` file. A route says: "if the request path matches this pattern, send it to this backend service."
- **Filters** are used to add behavior, like checking a header, adding authentication, or rewriting the path. Spring Cloud Gateway has many filters ready to use, like `AddRequestHeader` or `RewritePath`.
- **Global filters** apply to every route. These are often used for logging, security, and metrics.
- Spring Cloud Gateway works well with **Spring Cloud Netflix Eureka** or **Consul** for service discovery, so routes do not need hardcoded addresses.
- For rate limiting, Spring Cloud Gateway can use **Redis** with the `RequestRateLimiter` filter.
- For security, it is common to combine the gateway with **Spring Security** and **OAuth2 Resource Server**, so the gateway can validate JWT tokens before forwarding requests.

A simple route definition in `application.yml` might look like this:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
          filters:
            - StripPrefix=1
```

This means: any request starting with `/orders/` gets sent to the service called `ORDER-SERVICE`, and the `/orders` part is removed before forwarding.

---

## Interview Questions

1. **What is an API Gateway, and why do microservice systems use one?** _Hint: talk about a single entry point, and reducing duplicate code across services._
    
2. **How is an API Gateway different from a load balancer?** _Hint: a load balancer just spreads traffic across servers. A gateway does much more — authentication, routing, transformation, rate limiting, and more._
    
3. **What happens if the API Gateway goes down? How do you prevent this?** _Hint: talk about single point of failure, and running multiple gateway instances behind a load balancer._
    
4. **Should the backend services still check permissions, even if the gateway already does?** _Hint: yes — defense in depth. Never fully trust that the gateway is the only line of defense._
    
5. **How would you implement rate limiting at the gateway level?** _Hint: mention token bucket algorithm, and tools like Redis with Spring Cloud Gateway's `RequestRateLimiter`._
    
6. **How does Spring Cloud Gateway differ from the older Netflix Zuul?** _Hint: Spring Cloud Gateway is built on non-blocking WebFlux; Zuul (1.x) was blocking, built on Servlet APIs._
    
7. **How would you handle service discovery in a gateway, instead of hardcoding backend URLs?** _Hint: mention Eureka, Consul, or Kubernetes service discovery, combined with a `lb://` URI scheme._
    
8. **How can an API Gateway help prevent OWASP API security risks?** _Hint: centralized auth, rate limiting, input validation, and stopping old API versions from being exposed._
    

---

## References

- [Spring Cloud Gateway Documentation](https://docs.spring.io/spring-cloud-gateway/reference/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x00-header/)
- [Kong Gateway Documentation](https://docs.konghq.com/gateway/)
- [AWS API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [NGINX as an API Gateway](https://www.nginx.com/learn/api-gateway/)