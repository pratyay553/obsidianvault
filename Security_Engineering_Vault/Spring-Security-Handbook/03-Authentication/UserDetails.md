# UserDetails

`UserDetails` is the core interface that represents a user's security-relevant data within Spring Security. It is deliberately decoupled from your application's own `User`/entity class.

## Interface Definition

```java
public interface UserDetails extends Serializable {
    String getUsername();
    String getPassword();
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

| Method | Meaning |
|---|---|
| `getUsername()` | Unique identifier used to log in |
| `getPassword()` | Encoded (hashed) password — never plain text |
| `getAuthorities()` | Roles/permissions granted to the user (e.g. `ROLE_ADMIN`) |
| `isAccountNonExpired()` | `false` blocks login if the account has expired |
| `isAccountNonLocked()` | `false` blocks login if the account is locked |
| `isCredentialsNonExpired()` | `false` blocks login if the password has expired |
| `isEnabled()` | `false` blocks login if the account is disabled |

If any of the four boolean checks return `false`, `DaoAuthenticationProvider` throws the corresponding exception (`AccountExpiredException`, `LockedException`, `CredentialsExpiredException`, `DisabledException`) before password verification even completes.

## Built-in Implementation: `User`

Spring Security provides a ready-to-use implementation, `org.springframework.security.core.userdetails.User`, along with a builder:

```java
UserDetails user = User.builder()
        .username("jdoe")
        .password(passwordEncoder.encode("secret"))
        .roles("USER")
        .build();
```

## Custom Implementation (wrapping your own entity)

Most real applications adapt their own JPA entity to `UserDetails` rather than duplicating data:

```java
public class UserPrincipal implements UserDetails {

    private final AppUser appUser; // your own entity

    public UserPrincipal(AppUser appUser) {
        this.appUser = appUser;
    }

    @Override
    public String getUsername() { return appUser.getEmail(); }

    @Override
    public String getPassword() { return appUser.getPasswordHash(); }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return appUser.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toList());
    }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return !appUser.isLocked(); }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return appUser.isActive(); }
}
```

## Related

- `UserDetailsService.md` — loads `UserDetails` by username
- `AuthenticationProvider.md` — consumes `UserDetails` during authentication
- `PasswordEncoder.md` — verifies the password stored in `UserDetails`
