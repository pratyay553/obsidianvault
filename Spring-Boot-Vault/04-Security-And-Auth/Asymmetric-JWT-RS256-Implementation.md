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

# 📌 Asymmetric JWT (RS256) Implementation

## 💡 1-Sentence Elevator Pitch (For Interviews)
> RS256 JWT uses a private key to sign tokens and a public key to verify them — the auth service holds the private key (only it can issue tokens), any resource service can verify with the public key (no shared secret needed), making it the correct choice for multi-service architectures.

## 🧱 Key Concepts & Mechanics

### HS256 vs RS256

|                  | HS256 (Symmetric)                | RS256 (Asymmetric)                       |
| ---------------- | -------------------------------- | ---------------------------------------- |
| Algorithm        | HMAC-SHA256                      | RSA-SHA256                               |
| Sign key         | Shared secret                    | Private key                              |
| Verify key       | Same shared secret               | Public key (safe to distribute)          |
| Architecture fit | Single service                   | Multi-service / microservices            |
| Risk             | Secret exposure signs + verifies | Private key exposure only allows signing |

### JWT Structure
```
Header.Payload.Signature
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIiwiaWF0IjoxNjkwMDAwMDAwfQ.{RSA_signature}
```

### Claims (Standard)
| Claim | Meaning |
|-------|---------|
| `sub` | Subject (user ID) |
| `iat` | Issued at (epoch seconds) |
| `exp` | Expiration time |
| `iss` | Issuer |
| `aud` | Audience |
| `jti` | JWT ID (for revocation) |

### Key Generation (Production)
```bash
# Generate RSA key pair (2048-bit minimum, 4096 recommended for high-security)
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
# Store private.pem in secrets manager (Vault, AWS Secrets Manager)
# Distribute public.pem to resource services
```

## 💻 Production Code Example

```java
// ── application.yml ──────────────────────────────────────────────────
// auth:
//   jwt:
//     private-key: classpath:keys/private.pem
//     public-key: classpath:keys/public.pem
//     access-token-expiry: PT15M     # 15 minutes
//     issuer: https://auth.example.com

// ── Properties ────────────────────────────────────────────────────────
@ConfigurationProperties(prefix = "auth.jwt")
public record JwtProperties(
    RSAPrivateKey privateKey,
    RSAPublicKey publicKey,
    Duration accessTokenExpiry,
    String issuer
) {}

// ── JWT Service ───────────────────────────────────────────────────────
@Service
@RequiredArgsConstructor
public class JwtService {

    private final JwtProperties props;

    public String generateAccessToken(UserDetails user) {
        Instant now = Instant.now();
        List<String> roles = user.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .toList();

        return JWT.create()
            .withIssuer(props.issuer())
            .withSubject(user.getUsername())
            .withIssuedAt(now)
            .withExpiresAt(now.plus(props.accessTokenExpiry()))
            .withJWTId(UUID.randomUUID().toString())    // for revocation
            .withClaim("roles", roles)
            .sign(Algorithm.RSA256(null, props.privateKey()));
    }

    public DecodedJWT verifyAndDecode(String token) {
        return JWT.require(Algorithm.RSA256(props.publicKey(), null))
            .withIssuer(props.issuer())
            .build()
            .verify(token);   // throws JWTVerificationException on failure
    }

    public String extractUsername(String token) {
        return verifyAndDecode(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails user) {
        try {
            DecodedJWT decoded = verifyAndDecode(token);
            return decoded.getSubject().equals(user.getUsername());
        } catch (JWTVerificationException e) {
            return false;
        }
    }
}

// ── Spring Security OAuth2 Resource Server config (alternative approach) ──
// Use Spring's built-in JWT support instead of manual filter for resource servers
@Configuration
@EnableWebSecurity
public class ResourceServerConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(jwt -> jwt.decoder(jwtDecoder()))
            );
        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder(JwtProperties props) {
        // Spring's built-in RSA verifier — no third-party JWT lib needed
        return NimbusJwtDecoder
            .withPublicKey(props.publicKey())
            .signatureAlgorithm(SignatureAlgorithm.RS256)
            .build();
    }

    // Convert JWT claims to Spring Security authorities
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();
        converter.setAuthoritiesClaimName("roles");
        converter.setAuthorityPrefix("ROLE_");
        JwtAuthenticationConverter jwtConverter = new JwtAuthenticationConverter();
        jwtConverter.setJwtGrantedAuthoritiesConverter(converter);
        return jwtConverter;
    }
}

// ── Auth controller ───────────────────────────────────────────────────
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authManager;
    private final JwtService jwtService;
    private final RefreshTokenService refreshTokenService;

    @PostMapping("/login")
    public TokenResponse login(@Valid @RequestBody LoginRequest req) {
        Authentication auth = authManager.authenticate(
            new UsernamePasswordAuthenticationToken(req.username(), req.password())
        );
        UserDetails user = (UserDetails) auth.getPrincipal();
        String accessToken = jwtService.generateAccessToken(user);
        String refreshToken = refreshTokenService.create(user.getUsername());
        return new TokenResponse(accessToken, refreshToken,
            jwtService.getExpiry().getEpochSecond());
    }
}

public record LoginRequest(
    @NotBlank String username,
    @NotBlank String password
) {}

public record TokenResponse(
    String accessToken,
    String refreshToken,
    long expiresAt
) {}
```

## ⚠️ Common Pitfalls & Anti-Patterns

- **Mistake:** Using HS256 (shared secret) in a microservices setup — every service that verifies tokens also has the power to forge them.
  **Fix:** Use RS256 — distribute only the public key to verifying services. The private key lives exclusively in the auth service.

- **Mistake:** Long access token expiry (hours/days) without a revocation mechanism.
  **Fix:** Keep access tokens short-lived (5-15 minutes). Use refresh tokens (longer-lived, stored in DB) for renewal and implement a `jti` blocklist for revocation.

- **Mistake:** Storing the private key in `application.yml` and committing to git.
  **Fix:** Load from environment variable, HashiCorp Vault, AWS Secrets Manager, or Kubernetes Secret.

- **Mistake:** Not validating `iss` (issuer) and `aud` (audience) claims.
  **Fix:** Always verify issuer and audience in the `JWT.require()` chain to prevent token substitution attacks.

- **Mistake:** Putting sensitive data (passwords, PII) in JWT payload.
  **Fix:** JWTs are Base64-encoded — anyone can decode the payload. Put only non-sensitive identifiers (user ID, roles).

## ❓ Frequently Asked Interview Questions

1. **Q:** Why use RS256 over HS256?
   - **A:** RS256 uses a key pair — only the auth service with the private key can sign tokens, but any resource service can verify with the (safely distributable) public key. HS256 requires every verifying service to hold the same secret, meaning any service can forge tokens. RS256 is mandatory for multi-service architectures.

2. **Q:** What is a JWT and what are its three parts?
   - **A:** A JSON Web Token is a compact, URL-safe string with three dot-separated parts: **Header** (algorithm type), **Payload** (claims — sub, iat, exp, custom fields), and **Signature** (HMAC or RSA signature over header.payload). The signature ensures integrity — a tampered token fails verification.

3. **Q:** Where should you store JWTs on the frontend?
   - **A:** `HttpOnly` cookies (not accessible to JavaScript) are safest against XSS. `localStorage` is vulnerable to XSS attacks. If using `Authorization: Bearer`, use in-memory storage and never `localStorage` for high-security apps.

4. **Q:** How do you implement JWT revocation if tokens are stateless?
   - **A:** Include a `jti` (JWT ID) claim in every token. Maintain a `revoked_tokens` table/Redis set. At validation, check if the `jti` is in the blocklist. For access tokens with short TTL, revoke only refresh tokens and let access tokens naturally expire.

5. **Q:** What is token rotation in the context of refresh tokens?
   - **A:** When a refresh token is used, immediately invalidate it and issue a new one. If the old refresh token is used again (theft detection), invalidate the entire session. This limits the window of abuse if a refresh token is stolen.

## 🔗 Related Notes
- [[Spring-Security-Architecture-Filters]]
- [[Refresh-Tokens-And-Session-Management]]
- [[Method-Level-Security-PreAuthorize]]
