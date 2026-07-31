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

# 📌 Spring Security Architecture & Filters

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Spring Security is a chain of servlet filters where each request passes through in order — authentication filters extract credentials, the `AuthenticationManager` verifies them, and authorization filters check permissions — all configurable via a single `SecurityFilterChain` bean in Spring Boot 3.

## 🧱 Key Concepts & Mechanics

### Filter Chain Architecture
```
HTTP Request
  → DelegatingFilterProxy (bridges Servlet ↔ Spring)
    → FilterChainProxy (Spring Security's entry point)
      → SecurityFilterChain (your configured chain)
        → Filter 1: SecurityContextPersistenceFilter
        → Filter 2: UsernamePasswordAuthenticationFilter
        → Filter 3: BearerTokenAuthenticationFilter (OAuth2/JWT)
        → Filter 4: ExceptionTranslationFilter
        → Filter 5: AuthorizationFilter (was FilterSecurityInterceptor)
          → Your Controller
```

### Key Components

| Component | Role |
|-----------|------|
| `SecurityFilterChain` | Your configured filter chain (replaces `WebSecurityConfigurerAdapter`) |
| `AuthenticationManager` | Delegates to `AuthenticationProvider`s to verify credentials |
| `AuthenticationProvider` | Does the actual verification (DB lookup, JWT decode, LDAP) |
| `UserDetailsService` | Loads `UserDetails` by username for password-based auth |
| `SecurityContextHolder` | Thread-local store for the authenticated `Authentication` object |
| `GrantedAuthority` | A single permission/role string (e.g., `ROLE_ADMIN`) |
| `ExceptionTranslationFilter` | Converts `AuthenticationException` → 401, `AccessDeniedException` → 403 |

### Spring Boot 3 Config Style (`SecurityFilterChain` bean)

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        )
        .sessionManagement(sm -> sm
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        )
        .csrf(AbstractHttpConfigurer::disable)
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
    return http.build();
}
```

### Custom Filter Placement
- `http.addFilterBefore(myFilter, UsernamePasswordAuthenticationFilter.class)` — run before
- `http.addFilterAfter(myFilter, BearerTokenAuthenticationFilter.class)` — run after
- `http.addFilterAt(myFilter, TargetFilter.class)` — replace (same position)

## 💻 Production Code Example

```java
// ── Full JWT-based security config ──────────────────────────────────
@Configuration
@EnableWebSecurity
@EnableMethodSecurity           // enables @PreAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtFilter;
    private final CustomAuthenticationEntryPoint authEntryPoint;
    private final CustomAccessDeniedHandler accessDeniedHandler;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors(Customizer.withDefaults())
            .csrf(AbstractHttpConfigurer::disable)
            .sessionManagement(sm ->
                sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.POST, "/api/v1/auth/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/products/**").permitAll()
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(authEntryPoint)   // 401
                .accessDeniedHandler(accessDeniedHandler)   // 403
            );
        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}

// ── Custom JWT filter ────────────────────────────────────────────────
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                     HttpServletResponse res,
                                     FilterChain chain) throws ServletException, IOException {
        String authHeader = req.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(req, res);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtService.isTokenValid(token, userDetails)) {
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(req));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(req, res);
    }
}

// ── Custom entry points ──────────────────────────────────────────────
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest req, HttpServletResponse res,
                          AuthenticationException ex) throws IOException {
        res.setContentType(MediaType.APPLICATION_JSON_VALUE);
        res.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        res.getWriter().write("{\"error\":\"Unauthorized\",\"message\":\"" + ex.getMessage() + "\"}");
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using deprecated `WebSecurityConfigurerAdapter` (removed in Spring Security 6).
  **Fix:** Expose `SecurityFilterChain`, `AuthenticationManager`, and `PasswordEncoder` as `@Bean`s.

- **Mistake:** 401/403 errors not returned as JSON — default is HTML redirect to login page.
  **Fix:** Set `SessionCreationPolicy.STATELESS` and implement `AuthenticationEntryPoint` + `AccessDeniedHandler`.

- **Mistake:** CSRF enabled for a stateless JWT API — every non-GET request needs a CSRF token.
  **Fix:** `csrf(AbstractHttpConfigurer::disable)` for stateless APIs. Keep CSRF enabled for session-based apps.

- **Mistake:** `SecurityContextHolder.getContext().getAuthentication()` returning null outside a request thread.
  **Fix:** The security context is thread-local. For async methods, configure `SecurityContextHolder.setStrategyName(MODE_INHERITABLETHREADLOCAL)` or use `DelegatingSecurityContextExecutor`.

- **Mistake:** Storing user roles without `ROLE_` prefix and using `hasRole("ADMIN")`.
  **Fix:** `hasRole("ADMIN")` automatically prepends `ROLE_`. If storing `ADMIN` in DB, it maps correctly. If storing `ROLE_ADMIN`, use `hasAuthority("ROLE_ADMIN")`.

## ❓ Frequently Asked Interview Questions

1. **Q:** How does `DelegatingFilterProxy` work?
   - **A:** It's a standard servlet `Filter` registered in the web container that delegates to a Spring-managed bean named `springSecurityFilterChain`. This bridges the gap between the servlet container's filter lifecycle and Spring's application context.

2. **Q:** What is `OncePerRequestFilter` and why use it?
   - **A:** A Spring convenience class that guarantees the filter executes exactly once per request, even for async dispatches or forward/include dispatches. Custom security filters should extend it.

3. **Q:** What happens when an `AccessDeniedException` is thrown?
   - **A:** `ExceptionTranslationFilter` catches it. If the user is not authenticated, it triggers the `AuthenticationEntryPoint` (→ 401). If authenticated but lacking permissions, it calls the `AccessDeniedHandler` (→ 403).

4. **Q:** How do you secure specific HTTP methods differently (e.g., GET is public, POST requires auth)?
   - **A:** Use `requestMatchers(HttpMethod.GET, "/api/products/**").permitAll()` and `requestMatchers(HttpMethod.POST, "/api/products/**").authenticated()`. Order matters — more specific rules must come first.

## 🔗 Related Notes
- [[Asymmetric-JWT-RS256-Implementation]]
- [[Method-Level-Security-PreAuthorize]]
- [[CORS-And-Headers]]
- [[Refresh-Tokens-And-Session-Management]]
