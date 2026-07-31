# Custom Authentication

This document walks through building a fully custom authentication mechanism in Spring Security — useful for API keys, OTPs, third-party token validation, or any flow that doesn't fit username/password out of the box.

## When You Need This

- Authenticating via a header-based API key.
- Validating a token issued by an external identity provider.
- One-time-password (OTP) / SMS-based login.
- Legacy authentication systems that can't be adapted to `UserDetailsService`.

## The Pieces You'll Build

1. A custom `Authentication` token class (holds the credentials pre- and post-authentication).
2. A custom `AuthenticationProvider` (does the actual verification).
3. A custom filter (extracts credentials from the request and invokes the `AuthenticationManager`).
4. Security configuration wiring it all together.

## 1. Custom Authentication Token

```java
public class ApiKeyAuthenticationToken extends AbstractAuthenticationToken {

    private final String apiKey;
    private Object principal;

    // Pre-authentication constructor
    public ApiKeyAuthenticationToken(String apiKey) {
        super(null);
        this.apiKey = apiKey;
        setAuthenticated(false);
    }

    // Post-authentication constructor
    public ApiKeyAuthenticationToken(Object principal, String apiKey,
                                      Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.apiKey = apiKey;
        super.setAuthenticated(true);
    }

    @Override
    public Object getCredentials() { return apiKey; }

    @Override
    public Object getPrincipal() { return principal; }
}
```

## 2. Custom AuthenticationProvider

```java
@Component
public class ApiKeyAuthenticationProvider implements AuthenticationProvider {

    private final ApiKeyRepository apiKeyRepository;

    public ApiKeyAuthenticationProvider(ApiKeyRepository apiKeyRepository) {
        this.apiKeyRepository = apiKeyRepository;
    }

    @Override
    public Authentication authenticate(Authentication authentication) {
        String apiKey = (String) authentication.getCredentials();

        ApiKeyEntity entity = apiKeyRepository.findByKey(apiKey)
                .orElseThrow(() -> new BadCredentialsException("Invalid API key"));

        List<GrantedAuthority> authorities =
                List.of(new SimpleGrantedAuthority("ROLE_" + entity.getRole()));

        return new ApiKeyAuthenticationToken(entity.getOwner(), apiKey, authorities);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return ApiKeyAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

## 3. Custom Filter

```java
public class ApiKeyAuthenticationFilter extends OncePerRequestFilter {

    private final AuthenticationManager authenticationManager;

    public ApiKeyAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {

        String apiKey = request.getHeader("X-API-KEY");

        if (apiKey != null) {
            Authentication authRequest = new ApiKeyAuthenticationToken(apiKey);
            Authentication result = authenticationManager.authenticate(authRequest);
            SecurityContextHolder.getContext().setAuthentication(result);
        }

        chain.doFilter(request, response);
    }
}
```

## 4. Wiring It Together

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
            ApiKeyAuthenticationProvider apiKeyProvider) throws Exception {

        AuthenticationManager authManager = new ProviderManager(apiKeyProvider);

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated())
            .addFilterBefore(
                new ApiKeyAuthenticationFilter(authManager),
                UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

## Checklist for Any Custom Scheme

- [ ] Does `supports()` correctly match only your custom `Authentication` type?
- [ ] Does the provider throw a specific `AuthenticationException` subtype on failure (not a generic `RuntimeException`)?
- [ ] Is the filter positioned correctly relative to other filters in the chain (`addFilterBefore`/`addFilterAfter`)?
- [ ] Are credentials erased from the `Authentication` object after success, where sensitive?
- [ ] Have you disabled unneeded default mechanisms (form login, basic auth) if they conflict?

## Related

- `AuthenticationProvider.md` — the interface implemented above
- `AuthenticationManager.md` / `ProviderManager.md` — how the custom provider gets invoked
- `Authentication-Flow.md` — where this custom flow fits into the bigger picture
