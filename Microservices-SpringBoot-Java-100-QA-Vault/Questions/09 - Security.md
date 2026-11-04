---
tags: [microservices, springboot, java, interview, security]
category: Security
questions: 69-76
status: to-revise
---

# Security (Q69 to Q76)

Back to [[00 - Index]] | Words you don't know? See [[Glossary]]

> [!tip] How to use this note
> Read the question, **try to answer in your own words**, then click the question to open the answer. 🟢 = basic, 🟡 = medium, 🔴 = advanced.

> [!question]- Q69. What is the difference between authentication and authorization? 🟢
> - **Authentication (AuthN):** *Who are you?* (login, token check)
> - **Authorization (AuthZ):** *What are you allowed to do?* (roles, permissions)
>
> Example: showing your ID at the airport is authentication. Your ticket deciding which gate you can enter is authorization.
> HTTP: 401 = authentication problem, 403 = authorization problem.

> [!question]- Q70. What is a JWT and how is it validated? 🟡
> A **JWT (JSON Web Token)** is a signed token with three parts: `header.payload.signature`. The payload has **claims** (user id, roles, expiry). Services verify the **signature** with the identity provider's public key, and check the expiry, so they do not need to call the login server every time.
> Important: the payload is only **encoded, not encrypted**. Never put secrets in it. Keep expiry short.
>
> 📖 **Simple words:**
> - **Claim** = a piece of information inside the token (like `sub` = user id).
> - **Signature** = proof that the token was not changed.

> [!question]- Q71. What are OAuth2 and OpenID Connect (OIDC)? 🟡
> - **OAuth2:** a standard for giving an app **limited access** without sharing a password. (Authorization.)
> - **OIDC:** a layer on top of OAuth2 that also tells you **who the user is** (Authentication), using an ID token.
>
> Common flows: **Authorization Code + PKCE** for web/mobile users, **Client Credentials** for service-to-service calls.
> Popular identity providers: Keycloak, Auth0, Okta, Azure AD.
>
> 📖 **Simple words:**
> - **Identity provider (IdP)** = the system that logs users in and issues tokens.
> - **Flow (grant type)** = the steps used to get a token.

> [!question]- Q72. How do you secure a Spring Boot service with Spring Security and JWT? 🟡
> Make the service an **OAuth2 Resource Server**. Spring validates the JWT for you. Define a `SecurityFilterChain` bean (the old `WebSecurityConfigurerAdapter` is removed).
>
> 📖 **Simple words:**
> - **Resource server** = an API that accepts and checks tokens.
>
> ```java
> @Bean
> SecurityFilterChain security(HttpSecurity http) throws Exception {
>     return http
>         .csrf(csrf -> csrf.disable())                       // stateless token API
>         .authorizeHttpRequests(a -> a
>             .requestMatchers("/actuator/health/**").permitAll()
>             .requestMatchers(HttpMethod.POST, "/orders").hasAuthority("SCOPE_orders.write")
>             .anyRequest().authenticated())
>         .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
>         .build();
> }
> ```
> ```yaml
> spring.security.oauth2.resourceserver.jwt.issuer-uri: https://auth.example.com/realms/shop
> ```
> Store user passwords (if you manage them) only as hashes using `BCryptPasswordEncoder`, never plain text.

> [!question]- Q73. How do you secure service-to-service communication? 🔴
> - Use the **Client Credentials** flow: the calling service gets its own token and sends it.
> - Or **relay the user token** (token relay) when user identity matters downstream.
> - Encrypt traffic with **TLS**; use **mTLS** (often via a service mesh) so services prove identity.
> - Apply the **zero trust** idea: do not trust a call just because it comes from inside the network.
>
> 📖 **Simple words:**
> - **Zero trust** = every request must be verified, even from inside the network.

> [!question]- Q74. Should tokens be validated at the gateway or in each service? 🟡
> Best practice: **both**. The gateway rejects bad requests early (fast, less load). Each service still validates the token and checks permissions, because someone might call a service directly or a gateway rule may be wrong. This is called **defense in depth**.
>
> 📖 **Simple words:**
> - **Defense in depth** = several layers of security, so one failure is not a disaster.

> [!question]- Q75. What are CORS and CSRF? What other attacks should you know? 🟡
> - **CORS:** a browser rule that blocks a web page from calling another domain unless that server allows it. Configure allowed origins explicitly (not `*` for secured APIs).
> - **CSRF:** a trick where another site makes your browser send a request using your cookies. Needed for cookie/session apps; token-in-header APIs are usually safe.
> - Also know (OWASP Top 10): **SQL injection** (use parameterised queries/JPA), **XSS**, broken access control, sensitive data exposure, vulnerable dependencies.
>
> 📖 **Simple words:**
> - **Origin** = protocol + domain + port, like `https://app.example.com`.
> - **OWASP** = a group that publishes the most common web security risks.
>
> ```java
> @Bean
> CorsConfigurationSource cors() {
>     CorsConfiguration c = new CorsConfiguration();
>     c.setAllowedOrigins(List.of("https://app.example.com"));
>     c.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
>     UrlBasedCorsConfigurationSource s = new UrlBasedCorsConfigurationSource();
>     s.registerCorsConfiguration("/**", c);
>     return s;
> }
> ```

> [!question]- Q76. How do you restrict access by role inside a method? 🟡
> Enable **method security** and use `@PreAuthorize` with a rule. This is useful for fine-grained checks, like "an admin, or the owner of this record".
>
> ```java
> @Configuration
> @EnableMethodSecurity
> class MethodSecurityConfig {}
>
> @PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
> public Profile getProfile(String userId) { ... }
> ```

---
Previous: [[08 - Resilience and Distributed Data]] | Next: [[10 - Testing]]
