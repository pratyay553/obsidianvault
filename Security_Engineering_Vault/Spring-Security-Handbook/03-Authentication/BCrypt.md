# BCrypt

BCrypt is a password-hashing algorithm purpose-built to be slow and resistant to brute-force and rainbow-table attacks. Spring Security's `BCryptPasswordEncoder` is the most commonly used `PasswordEncoder` implementation.

## Why BCrypt?

- **Adaptive cost** — a configurable "strength" (work factor) makes hashing deliberately slow, and can be increased over time as hardware gets faster.
- **Built-in salting** — a random salt is generated per password and stored as part of the hash itself, so identical passwords never produce identical hashes.
- **Battle-tested** — based on the Blowfish cipher, in wide use since 1999.

## Anatomy of a BCrypt Hash

```
$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
└┬┘└┬┘└──────────┬───────────┘└───────────┬──────────────────┘
 │  │            │                        │
 │  │            │                        └─ 31-char hash
 │  │            └─ 22-char salt
 │  └─ cost factor (work factor), e.g. 10 = 2^10 rounds
 └─ algorithm version identifier
```

- **Version** (`$2a$`, `$2b$`, `$2y$`) — identifies the BCrypt variant used.
- **Cost factor** — number of hashing rounds is `2^cost`. Default in Spring Security is `10`; each +1 doubles the computation time.
- **Salt** — 16 random bytes, base64-encoded, unique per password.
- **Hash** — the actual output.

## Using BCryptPasswordEncoder

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(); // default strength = 10
}

// Custom strength (higher = slower/more secure, typical range 10-12)
new BCryptPasswordEncoder(12);
```

```java
String hash = passwordEncoder.encode("mySecret123");
// => $2a$10$7EqJtq98hPqEX7fNZaFWoO...

boolean matches = passwordEncoder.matches("mySecret123", hash); // true
```

## Choosing a Cost Factor

| Cost | Approx. time per hash* | Notes |
|---|---|---|
| 10 | ~65ms | Spring Security default; reasonable for most web apps |
| 12 | ~250ms | Higher security, noticeable latency on login |
| 14+ | ~1s+ | Rarely needed outside high-security contexts |

*Timings vary significantly by hardware; benchmark on your target infrastructure.

The cost factor should be tuned so hashing takes roughly 100–300ms on your production hardware — slow enough to deter brute-force attempts, fast enough not to degrade user experience.

## Key Takeaways

- BCrypt hashes are **self-contained**: salt and cost factor travel with the hash, so no separate salt storage is needed.
- Never truncate BCrypt's 60-character output when defining a database column (`VARCHAR(60)` minimum).
- BCrypt has a **72-byte password length limit** — longer inputs are silently truncated by the underlying algorithm; pre-hashing with SHA-256 first is a common mitigation if very long passwords must be supported.

## Related

- `PasswordEncoder.md` — the interface BCrypt implements
- `AuthenticationProvider.md` — invokes password matching during login
