# AuthenticationProvider

`AuthenticationProvider` is the interface that performs the actual work of validating a specific type of `Authentication`.

## Interface Definition

```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
    boolean supports(Class<?> authentication);
}
```

- **`authenticate(...)`** — performs the real authentication logic and returns a fully populated `Authentication` on success.
- **`supports(Class<?>)`** — tells `ProviderManager` whether this provider can handle a given `Authentication` implementation, so `ProviderManager` knows which provider(s) to try.

## Most Common Implementation: DaoAuthenticationProvider

`DaoAuthenticationProvider` is the built-in provider used for standard username/password authentication:

```java
DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
provider.setUserDetailsService(userDetailsService);
provider.setPasswordEncoder(passwordEncoder);
```

Its `authenticate()` method:
1. Loads the user via `UserDetailsService.loadUserByUsername(username)`.
2. Verifies the raw password against the stored hash using `PasswordEncoder.matches(...)`.
3. Runs additional checks (account not locked, not expired, not disabled) via `UserDetailsChecker`s.
4. Returns a new `UsernamePasswordAuthenticationToken` populated with the user's authorities, marked as authenticated.

## Writing a Custom AuthenticationProvider

```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication)
            throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();

        if (!isValid(username, password)) {
            throw new BadCredentialsException("Invalid username or password");
        }

        return new UsernamePasswordAuthenticationToken(
                username,
                password,
                List.of(new SimpleGrantedAuthority("ROLE_USER")));
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }

    private boolean isValid(String username, String password) {
        // custom validation logic — external API, legacy system, etc.
        return true;
    }
}
```

Register it alongside (or instead of) `DaoAuthenticationProvider` in a `ProviderManager`.

## Other Built-in Providers

| Provider | Use Case |
|---|---|
| `DaoAuthenticationProvider` | Username/password against a `UserDetailsService` |
| `LdapAuthenticationProvider` | LDAP directory authentication |
| `JwtAuthenticationProvider` | Validating JWT tokens (Resource Server) |
| `RememberMeAuthenticationProvider` | Remember-me tokens |
| `PreAuthenticatedAuthenticationProvider` | Trusting an upstream system (e.g. SSO header) |

## Related

- `ProviderManager.md` — orchestrates multiple providers
- `UserDetailsService.md`, `UserDetails.md`, `PasswordEncoder.md` — collaborators used by `DaoAuthenticationProvider`
- `Custom-Authentication.md` — a full custom authentication walkthrough
