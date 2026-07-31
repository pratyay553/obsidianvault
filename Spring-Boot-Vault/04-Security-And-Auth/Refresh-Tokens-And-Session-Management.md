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

# 📌 Refresh Tokens & Session Management

## 💡 1-Sentence Elevator Pitch (For Interviews)
> Refresh tokens are long-lived, single-use credentials stored server-side that allow clients to obtain new short-lived access tokens without re-authenticating — rotation invalidates the refresh token on each use, and reuse detection (using the same token twice) triggers full session revocation.

## 🧱 Key Concepts & Mechanics

### Token Pair Pattern
- **Access Token** (JWT): Short-lived (5–15 min), stateless, sent in `Authorization: Bearer` header
- **Refresh Token**: Long-lived (7–30 days), opaque random string, stored in DB, sent in `HttpOnly` cookie or body

### Refresh Token Lifecycle
1. Login → issue access token + refresh token → store refresh token hash in DB
2. Access token expires → client sends refresh token
3. Server validates token → marks old token as used → issues new access token + new refresh token (rotation)
4. Logout → delete refresh token from DB (immediate revocation)
5. Reuse detected (token already marked used) → revoke entire family (all tokens for that session)

### Token Family / Reuse Detection
- Assign a `familyId` to each login session
- When rotation happens, the new token inherits the same `familyId`
- If an already-used token arrives, the whole family is revoked → user must re-login

### Storage Options for Refresh Tokens
- PostgreSQL — strong consistency, easy to query and expire
- Redis — fast lookup, built-in TTL expiry, scales well
- Both: store in DB for audit + Redis for fast validation

## 💻 Production Code Example

```java
// ── DB Schema ────────────────────────────────────────────────────────
@Entity
@Table(name = "refresh_tokens")
public class RefreshToken {

    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(unique = true, nullable = false)
    private String tokenHash;           // SHA-256 hash of the raw token

    @Column(nullable = false)
    private String username;

    @Column(nullable = false)
    private UUID familyId;              // groups tokens for one login session

    @Column(nullable = false)
    private boolean used;               // true after rotation

    @Column(nullable = false)
    private Instant expiresAt;

    private Instant createdAt;
    private String userAgent;           // for audit
    private String ipAddress;
}

// ── Service ────────────────────────────────────────────────────────────
@Service
@Transactional
@RequiredArgsConstructor
public class RefreshTokenService {

    private final RefreshTokenRepository repo;
    private final JwtService jwtService;

    public String create(String username) {
        return create(username, UUID.randomUUID()); // new session family
    }

    private String create(String username, UUID familyId) {
        String rawToken = generateSecureToken();    // 32-byte random URL-safe string
        String hash = sha256(rawToken);
        RefreshToken entity = new RefreshToken();
        entity.setTokenHash(hash);
        entity.setUsername(username);
        entity.setFamilyId(familyId);
        entity.setUsed(false);
        entity.setExpiresAt(Instant.now().plus(Duration.ofDays(30)));
        entity.setCreatedAt(Instant.now());
        repo.save(entity);
        return rawToken;   // raw token returned to client, only hash stored in DB
    }

    public TokenPair rotate(String rawToken) {
        String hash = sha256(rawToken);
        RefreshToken token = repo.findByTokenHash(hash)
            .orElseThrow(() -> new AuthException("Invalid refresh token"));

        if (token.isUsed()) {
            // THEFT DETECTED — revoke entire family
            repo.deleteByFamilyId(token.getFamilyId());
            throw new AuthException("Refresh token reuse detected — session revoked");
        }

        if (token.getExpiresAt().isBefore(Instant.now())) {
            repo.delete(token);
            throw new AuthException("Refresh token expired");
        }

        // Mark current token as used (rotation)
        token.setUsed(true);
        repo.save(token);

        // Issue new access + refresh token in same family
        String newAccess = jwtService.generateAccessToken(token.getUsername());
        String newRefresh = create(token.getUsername(), token.getFamilyId());
        return new TokenPair(newAccess, newRefresh);
    }

    public void revoke(String rawToken) {
        String hash = sha256(rawToken);
        repo.findByTokenHash(hash).ifPresent(t -> {
            repo.deleteByFamilyId(t.getFamilyId()); // revoke whole family on logout
        });
    }

    // Scheduled cleanup — runs daily
    @Scheduled(cron = "0 0 2 * * *")
    public void deleteExpiredTokens() {
        repo.deleteByExpiresAtBefore(Instant.now());
    }

    private String generateSecureToken() {
        byte[] bytes = new byte[32];
        new SecureRandom().nextBytes(bytes);
        return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
    }

    private String sha256(String input) {
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            return HexFormat.of().formatHex(digest.digest(input.getBytes(StandardCharsets.UTF_8)));
        } catch (NoSuchAlgorithmException e) {
            throw new IllegalStateException(e);
        }
    }
}

public record TokenPair(String accessToken, String refreshToken) {}

// ── Controller ─────────────────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final RefreshTokenService refreshTokenService;

    @PostMapping("/refresh")
    public TokenResponse refresh(@RequestBody RefreshRequest req) {
        TokenPair pair = refreshTokenService.rotate(req.refreshToken());
        return new TokenResponse(pair.accessToken(), pair.refreshToken());
    }

    @PostMapping("/logout")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void logout(@RequestBody LogoutRequest req) {
        refreshTokenService.revoke(req.refreshToken());
    }
}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Storing raw refresh tokens in the database.
  **Fix:** Store a SHA-256 hash. If the DB is compromised, raw tokens are useless to the attacker.

- **Mistake:** Not implementing token rotation — reusing the same refresh token indefinitely.
  **Fix:** Rotate on every use. Issue a new refresh token, mark the old one as used. Reuse of a spent token signals theft.

- **Mistake:** Storing refresh tokens in `localStorage` — accessible to XSS scripts.
  **Fix:** Store in `HttpOnly`, `Secure`, `SameSite=Strict` cookies. They're inaccessible to JavaScript.

- **Mistake:** Not cleaning up expired tokens — table grows unbounded.
  **Fix:** `@Scheduled` cleanup job or rely on DB TTL (Redis auto-expiry, PostgreSQL pg_cron).

- **Mistake:** Long-lived access tokens to avoid refresh complexity.
  **Fix:** Keep access tokens short (≤15 min). The refresh mechanism exists specifically to avoid long-lived access tokens.

## ❓ Frequently Asked Interview Questions

1. **Q:** Why do we need refresh tokens if we have access tokens?
   - **A:** Access tokens are short-lived (5-15 min) to limit damage if stolen — but re-entering credentials every 15 minutes is unusable. Refresh tokens allow silent renewal without re-authentication, while keeping the attack surface of access tokens small.

2. **Q:** What is refresh token rotation and why does it matter?
   - **A:** On each refresh, the old token is invalidated and a new one is issued. If the old token is used again (reuse detection), it signals theft — the entire session is revoked. Without rotation, a stolen refresh token grants indefinite access.

3. **Q:** How do you implement logout with JWTs?
   - **A:** Stateless access tokens can't be "invalidated" — they remain valid until expiry. The practical approach: delete the refresh token from DB (prevents renewal), optionally add the access token's `jti` to a short-lived blocklist (Redis) until its natural expiry, and clear cookies on the client.

4. **Q:** What is a token family?
   - **A:** A UUID that groups all refresh tokens issued within a single login session. When rotation occurs, the new token inherits the `familyId`. If a stolen token is reused, the `familyId` allows revoking every token in that session at once.

## 🔗 Related Notes
- [[Asymmetric-JWT-RS256-Implementation]]
- [[Spring-Security-Architecture-Filters]]
- [[Redis-Caching-Strategies]]
