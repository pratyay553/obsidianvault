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

# 📌 CORS & Security Headers

## 💡 1-Sentence Elevator Pitch (For Interviews)
> CORS (Cross-Origin Resource Sharing) is a browser security mechanism that blocks frontend JavaScript from calling APIs on a different origin — Spring Boot lets you configure allowed origins, methods, and headers either globally via `WebMvcConfigurer` or per-endpoint with `@CrossOrigin`.

## 🧱 Key Concepts & Mechanics

### How CORS Works
1. Browser sends a **preflight** `OPTIONS` request with `Origin` + `Access-Control-Request-Method`
2. Server responds with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, etc.
3. If the preflight succeeds, browser sends the real request
4. Failure → browser blocks the response (server still processed it)

### Spring CORS Configuration Points

| Approach | Scope | Use When |
|----------|-------|----------|
| `WebMvcConfigurer#addCorsMappings` | Global | Same policy across all endpoints |
| `@CrossOrigin` on controller/method | Per-endpoint | Fine-grained control |
| `CorsConfigurationSource` in `HttpSecurity` | Security layer | Spring Security is active |

### Important Security Headers

| Header | Purpose |
|--------|---------|
| `Strict-Transport-Security` | Force HTTPS (HSTS) |
| `X-Content-Type-Options: nosniff` | Prevent MIME sniffing |
| `X-Frame-Options: DENY` | Prevent clickjacking |
| `Content-Security-Policy` | Control resource origins |
| `Cache-Control: no-store` | Prevent caching sensitive responses |

Spring Security adds `X-Content-Type-Options`, `X-Frame-Options`, and `Cache-Control` by default.

## 💻 Production Code Example

```java
// ── Global CORS (without Spring Security) ────────────────────────────
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins(
                "https://app.example.com",
                "https://admin.example.com"
            )
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
            .allowedHeaders("Authorization", "Content-Type", "X-Request-ID")
            .exposedHeaders("X-Total-Count", "Location")
            .allowCredentials(true)    // required for cookies/auth headers
            .maxAge(3600);             // preflight cache duration in seconds
    }
}

// ── CORS with Spring Security (required when Spring Security is active) ─
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
                                            CorsConfigurationSource corsSource) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsSource))
            .csrf(AbstractHttpConfigurer::disable)  // disable for stateless JWT APIs
            // ... other security config
        ;
        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://app.example.com"));
        config.setAllowedMethods(List.of("GET","POST","PUT","PATCH","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }
}

// ── Per-endpoint override ─────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/public")
@CrossOrigin(origins = "*", maxAge = 3600)  // public endpoints — any origin
public class PublicController { ... }

// ── Custom response headers filter ───────────────────────────────────
@Component
public class SecurityHeadersFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
                          FilterChain chain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("X-Content-Type-Options", "nosniff");
        response.setHeader("X-Frame-Options", "DENY");
        response.setHeader("Referrer-Policy", "strict-origin-when-cross-origin");
        chain.doFilter(req, res);
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Configuring CORS in `WebMvcConfigurer` when Spring Security is active — Security's filter runs first and blocks the preflight before MVC sees it.
  **Fix:** Always configure CORS via `CorsConfigurationSource` bean + `http.cors(...)` when Spring Security is present.

- **Mistake:** Using `allowedOrigins("*")` with `allowCredentials(true)` — browsers reject this combination.
  **Fix:** Specify explicit origins when credentials are required: `allowedOrigins("https://app.example.com")`.

- **Mistake:** Forgetting to expose custom response headers (`X-Total-Count`, `Location`) — browsers block access to non-exposed headers.
  **Fix:** Add headers to `exposedHeaders()` in the CORS config.

- **Mistake:** Disabling CSRF for session-based apps.
  **Fix:** Only disable CSRF for stateless JWT APIs. Session-based apps need CSRF protection.

## ❓ Frequently Asked Interview Questions

1. **Q:** What is CORS and why does it exist?
   - **A:** CORS is a browser security policy that restricts JavaScript running at `origin-a.com` from making requests to `api-b.com`. It exists to prevent malicious sites from silently reading data from APIs that trust the user's cookies. The restriction is browser-enforced — server-to-server calls are unaffected.

2. **Q:** What is a preflight request?
   - **A:** An `OPTIONS` request the browser automatically sends before a cross-origin request that uses non-simple methods (PUT, DELETE, PATCH) or custom headers. The server must respond with the appropriate `Access-Control-*` headers or the browser aborts the real request.

3. **Q:** Why can't you use `allowedOrigins("*")` with `allowCredentials(true)`?
   - **A:** Allowing any origin to send credentials (cookies, auth headers) would let malicious sites make authenticated requests on behalf of the user — defeating the purpose of CORS. Browsers explicitly reject this combination.

4. **Q:** Does CORS prevent the server from receiving the request?
   - **A:** No. CORS is purely browser-enforced. The server processes the request and returns a response regardless; the browser decides whether to expose that response to the JavaScript caller. Non-browser clients (curl, Postman, backend services) are unaffected.

## 🔗 Related Notes
- [[Spring-Security-Architecture-Filters]]
- [[Controllers-And-Request-Mapping]]
- [[RESTful-API-Design]]
