# UserDetailsService

`UserDetailsService` is a single-method strategy interface used to look up user data during authentication.

## Interface Definition

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

It has exactly one job: given a username, return a matching `UserDetails`, or throw `UsernameNotFoundException` if none exists.

## Where It's Called From

`DaoAuthenticationProvider` calls `loadUserByUsername(...)` as the first step of authentication, before any password check happens.

## Typical Implementation (Database-backed)

```java
@Service
public class AppUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public AppUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        AppUser appUser = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException(
                        "No user found with username: " + username));

        return new UserPrincipal(appUser); // implements UserDetails
    }
}
```

## Registering It

Spring Boot auto-detects a single `UserDetailsService` bean and wires it into the default `DaoAuthenticationProvider`:

```java
@Bean
public UserDetailsService userDetailsService(UserRepository repo) {
    return new AppUserDetailsService(repo);
}
```

## In-Memory Variant (useful for demos/tests)

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user = User.withUsername("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();

    return new InMemoryUserDetailsManager(user);
}
```

## Important Notes

- `UsernameNotFoundException` should be thrown for "user not found" rather than returning `null`, to avoid `NullPointerException`s further down the chain and to keep error handling consistent.
- To avoid leaking whether a username exists (a timing/enumeration attack), some implementations deliberately perform a dummy password comparison even when the user isn't found — Spring Security's `DaoAuthenticationProvider` already does this via a hidden "user not found" encoded password comparison.
- Don't confuse this with `UserDetailsManager` (a sub-interface adding create/update/delete operations for managing users, used by `InMemoryUserDetailsManager` and `JdbcUserDetailsManager`).

## Related

- `UserDetails.md` — the object this service returns
- `AuthenticationProvider.md` — the consumer of this service
- `PasswordEncoder.md` — used right after this service returns the stored hash
