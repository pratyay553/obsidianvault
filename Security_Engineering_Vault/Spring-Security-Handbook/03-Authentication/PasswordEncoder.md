# PasswordEncoder

`PasswordEncoder` is the strategy interface Spring Security uses to hash passwords and verify raw input against stored hashes.

## Interface Definition

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
    default boolean upgradeEncoding(String encodedPassword) { return false; }
}
```

| Method | Purpose |
|---|---|
| `encode(raw)` | Hashes a raw password for storage |
| `matches(raw, encoded)` | Verifies a raw password against a stored hash — **never decrypts**, since hashing is one-way |
| `upgradeEncoding(encoded)` | Signals the stored hash should be re-encoded (e.g. cost factor increased) |

## Why Passwords Are Never Compared Directly

Passwords are hashed, not encrypted — there is no way to reverse a hash back to the original password. `matches()` re-hashes the raw input (using the same salt/parameters embedded in the stored value) and compares the result, rather than "decrypting" anything.

## Recommended Implementation: BCryptPasswordEncoder

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

See `BCrypt.md` for details on how it works internally.

## DelegatingPasswordEncoder (Spring Boot Default)

Since Spring Security 5, `PasswordEncoderFactories.createDelegatingPasswordEncoder()` is the default. It prefixes stored hashes with an identifier (e.g. `{bcrypt}`, `{noop}`, `{pbkdf2}`) so multiple encoding schemes can coexist — useful when migrating from a weaker algorithm to a stronger one without invalidating existing hashes.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

Example stored value: `{bcrypt}$2a$10$dXj3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG`

## Usage in Authentication

```java
// Registration
String hashed = passwordEncoder.encode(rawPassword);
userRepository.save(new AppUser(username, hashed));

// Login (performed internally by DaoAuthenticationProvider)
boolean valid = passwordEncoder.matches(submittedRawPassword, storedHash);
```

## Common Mistakes to Avoid

- **Never store raw passwords** or use `NoOpPasswordEncoder` in production.
- **Never write your own hashing algorithm** — use a vetted implementation (BCrypt, PBKDF2, SCrypt, Argon2).
- **Never compare hashes with `String.equals()`** or by hashing manually — always go through `matches()`, which handles salt extraction and timing-safe comparison correctly.

## Related

- `BCrypt.md` — the most common concrete `PasswordEncoder`
- `AuthenticationProvider.md` — calls `matches()` during authentication
- `UserDetails.md` — holds the encoded password that gets compared
