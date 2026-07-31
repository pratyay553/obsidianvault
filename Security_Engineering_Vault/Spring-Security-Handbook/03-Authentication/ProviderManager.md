# ProviderManager

`ProviderManager` is Spring Security's default implementation of `AuthenticationManager`. Rather than authenticating credentials itself, it **delegates to a chain of `AuthenticationProvider` instances**.

## Why It Exists

Applications often need to support more than one way of authenticating (username/password, LDAP, pre-authenticated headers, OAuth2, custom tokens). `ProviderManager` allows each mechanism to be encapsulated in its own `AuthenticationProvider`, and tries each one in turn.

## How It Works

```java
public Authentication authenticate(Authentication authentication) {
    for (AuthenticationProvider provider : providers) {
        if (!provider.supports(authentication.getClass())) {
            continue;
        }
        try {
            Authentication result = provider.authenticate(authentication);
            if (result != null) {
                return result; // success — stop iterating
            }
        } catch (AuthenticationException e) {
            lastException = e; // remember and try next provider
        }
    }
    throw lastException; // no provider could authenticate
}
```

1. Iterates its configured list of `AuthenticationProvider`s in order.
2. For each provider, checks `supports(authentication.getClass())` — skips it if it can't handle this `Authentication` type.
3. Calls `provider.authenticate(...)`. If it returns a non-null result, authentication succeeded and iteration stops.
4. If a provider throws, `ProviderManager` remembers the exception and tries the next provider.
5. If no provider succeeds, the last exception is re-thrown.

## Parent AuthenticationManager

`ProviderManager` can also be configured with a **parent** `AuthenticationManager`, consulted if none of its own providers can authenticate the request. This is how global and per-`HttpSecurity` configurations can share a common fallback.

## Example Configuration

```java
@Bean
public AuthenticationManager authenticationManager(
        DaoAuthenticationProvider daoProvider,
        LdapAuthenticationProvider ldapProvider) {

    ProviderManager manager =
        new ProviderManager(List.of(daoProvider, ldapProvider));

    manager.setEraseCredentialsAfterAuthentication(true);
    return manager;
}
```

## Key Points

- `ProviderManager` is a **coordinator**, not an authenticator — actual verification logic lives in `AuthenticationProvider` implementations.
- Order of providers matters: the first one that `supports()` the `Authentication` type and doesn't throw wins.
- By default, credentials are erased from the returned `Authentication` object after a successful authentication (`eraseCredentialsAfterAuthentication`), for security hygiene.

## Related

- `AuthenticationManager.md` — the interface `ProviderManager` implements
- `AuthenticationProvider.md` — the pluggable strategies it delegates to
