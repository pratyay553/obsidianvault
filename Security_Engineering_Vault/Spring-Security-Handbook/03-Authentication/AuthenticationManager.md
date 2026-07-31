# AuthenticationManager

`AuthenticationManager` is the core Spring Security interface responsible for processing an authentication request.

## Interface Definition

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

## Responsibility

- Accepts an **unauthenticated** `Authentication` object (containing raw credentials).
- Returns a **fully authenticated** `Authentication` object (with granted authorities) on success.
- Throws an `AuthenticationException` subtype (e.g. `BadCredentialsException`, `DisabledException`, `LockedException`) on failure.

## Contract Rules

| Outcome | Behavior |
|---|---|
| Successful authentication | Returns `Authentication` with `isAuthenticated() == true` |
| Failed authentication | Throws `AuthenticationException` |
| Unable to decide | May return `null` (rare; most implementations throw instead) |

## Typical Implementation

Spring Security ships one primary implementation: **`ProviderManager`**, which delegates to a list of `AuthenticationProvider` beans rather than performing authentication itself. See `ProviderManager.md` for details.

## Where It's Used

Authentication filters (e.g. `UsernamePasswordAuthenticationFilter`, custom JWT filters) obtain an `AuthenticationManager` — usually via `AuthenticationConfiguration` in Spring Boot — and call `authenticate()` with the credentials extracted from the request.

```java
Authentication authRequest =
    new UsernamePasswordAuthenticationToken(username, password);

Authentication authResult = authenticationManager.authenticate(authRequest);

SecurityContextHolder.getContext().setAuthentication(authResult);
```

## Configuring a Custom AuthenticationManager

```java
@Bean
public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config) throws Exception {
    return config.getAuthenticationManager();
}
```

Or manually composed:

```java
@Bean
public AuthenticationManager authenticationManager(
        UserDetailsService userDetailsService,
        PasswordEncoder passwordEncoder) {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
    provider.setUserDetailsService(userDetailsService);
    provider.setPasswordEncoder(passwordEncoder);
    return new ProviderManager(provider);
}
```

## Related

- `ProviderManager.md` — the standard delegating implementation
- `AuthenticationProvider.md` — the pluggable strategy `ProviderManager` delegates to
