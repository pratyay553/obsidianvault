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

# 📌 Auth Service Architecture — Elevator Pitch

> Structured talk-tracks for explaining your auth system design in interviews. Adapt to the level of detail the interviewer is probing.

---

## ⚡ 30-Second Version (Screen / Coffee Chat)

> "I built a standalone auth service using Spring Boot. It issues RS256 JWTs — private key signs tokens, public key verifies. Access tokens expire in 15 minutes; refresh tokens live 30 days but are single-use and rotate on every call. The auth service exposes a JWKS endpoint, and resource services fetch the public key at startup — no DB call on every request. Refresh token theft is detected by family tracking: reuse of an already-rotated token revokes the entire session."

---

## 🎯 5-Minute Deep Dive (Technical Interview)

### Why RS256 over HS256?
> "HS256 uses a shared secret — any service that can verify tokens can also forge them. RS256 uses an asymmetric key pair: only the auth service holds the private key and can mint tokens; resource services have the public key and can only verify. In a microservices environment where tokens pass through multiple services, this isolation is non-negotiable."

### Token Design
> "Two-token model. Access tokens are short-lived JWTs — 15 minutes, stateless, no database lookup on verification. They carry `sub` (user ID), `roles`, `iss`, `aud`, `exp`, and `jti` (unique ID for revocation). Resource services verify the signature and claims locally using the public key.
>
> Refresh tokens are opaque random 32-byte strings. We store only the SHA-256 hash in PostgreSQL — if the DB is breached, raw tokens are useless. They expire in 30 days."

### Refresh Token Rotation & Theft Detection
> "On every refresh: the old token is marked as used and a new token pair is issued. The new refresh token inherits the same `familyId` as the session it came from. If an already-used token arrives — meaning someone is replaying a stolen token — we revoke every token in that family. The legitimate user is logged out and must re-authenticate, which is a small UX cost compared to an indefinite breach window."

### JWKS Endpoint
> "The auth service exposes `GET /.well-known/jwks.json` with the public RSA key in JWK format. Resource services fetch this on startup and cache it. They also poll periodically (every 12 hours) to support key rotation without redeployment. Spring Security's `NimbusJwtDecoder.withJwkSetUri()` handles this automatically."

### Security Headers & Storage
> "Refresh tokens go in `HttpOnly; Secure; SameSite=Strict` cookies — invisible to JavaScript, preventing XSS theft. Access tokens live in memory (not `localStorage`) on the frontend. On the server, we never log token values; all audit logs use the `jti`."

### Revocation
> "Access tokens are stateless — you can't truly revoke them before expiry. We accept this: 15 minutes is an acceptable staleness window. For immediate revocation (account lockout, password change), we add the `jti` to a Redis blocklist with a TTL matching the token's remaining lifetime. Resource services check the blocklist on sensitive operations. This is a deliberate tradeoff: Redis adds one fast lookup per request vs. the complexity of fully stateful tokens."

### Key Rotation
> "Private keys are stored in HashiCorp Vault, injected as environment secrets at startup. To rotate: generate a new key pair, update the JWKS endpoint to include both old and new public keys (overlap period covers existing valid tokens), update the signing key in Vault, and remove the old public key after the old tokens' max TTL has passed. No downtime, no forced logout."

### Database Schema
```sql
-- Users
CREATE TABLE users (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username    VARCHAR(50)  NOT NULL UNIQUE,
    email       VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(60) NOT NULL,        -- BCrypt, 12 rounds
    roles       TEXT[]       NOT NULL DEFAULT '{ROLE_USER}',
    locked      BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- Refresh tokens (only hashes stored)
CREATE TABLE refresh_tokens (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    token_hash  VARCHAR(64)  NOT NULL UNIQUE,  -- SHA-256 hex
    user_id     UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    family_id   UUID         NOT NULL,          -- revoke entire session on theft
    used        BOOLEAN      NOT NULL DEFAULT FALSE,
    expires_at  TIMESTAMPTZ  NOT NULL,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    ip_address  INET,
    user_agent  TEXT
);
CREATE INDEX idx_refresh_tokens_family ON refresh_tokens(family_id);
CREATE INDEX idx_refresh_tokens_expires ON refresh_tokens(expires_at)
    WHERE NOT used;   -- partial index — only active tokens
```

### Endpoints Summary
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/register` | None | Create account |
| POST | `/auth/login` | None | Issue token pair |
| POST | `/auth/refresh` | Refresh token | Rotate tokens |
| POST | `/auth/logout` | Refresh token | Revoke session |
| GET  | `/.well-known/jwks.json` | None | Public key for resource servers |
| POST | `/auth/password/change` | Access token | Change password + revoke all sessions |
| POST | `/auth/password/reset` | Email link | Reset forgotten password |

### Rate Limiting
> "Login endpoint is rate-limited by IP via Spring Cloud Gateway: 5 attempts per minute. After 5 failures, the account is temporarily locked (`locked = true`) for 15 minutes, preventing credential stuffing even if the attacker rotates IPs."

---

## 🔍 Follow-Up Questions & Answers

**Q: What if the private key is compromised?**
> Immediately rotate to a new key pair. Invalidate all existing refresh tokens (delete the `refresh_tokens` table rows). All sessions are terminated. This is an incident response procedure, not a normal flow.

**Q: How do you handle multiple devices / sessions?**
> Each login creates a new `familyId`. A user can have N concurrent families (N sessions). Logout revokes one family. "Logout all devices" deletes all rows for that `user_id`.

**Q: How do resource services get the user context?**
> The gateway validates the access token and forwards claims as trusted headers: `X-User-Id`, `X-User-Roles`. Downstream services trust these headers (protected by network policy — only the gateway can set them). No repeated JWT verification per service hop.

**Q: How do you scale the auth service horizontally?**
> Auth service is fully stateless — the private key is the only shared state, injected from Vault. Any instance can handle any request. DB handles refresh token state; Redis handles the JWT revocation blocklist. Horizontal scaling is straightforward.

**Q: What is the password hashing strategy?**
> BCrypt with cost factor 12 (Spring's `BCryptPasswordEncoder(12)`). At 12 rounds, a single hash takes ~500ms on modern hardware — acceptable for login, prohibitive for brute force. Upgrade path: re-hash on login if the stored hash was created at a lower factor.

**Q: How do you support social login (Google, GitHub)?**
> Separate OAuth2 callback flow. After verifying the OAuth2 code, we look up or create a local user account keyed by the external `sub` claim, then issue our own JWT pair. The user's session is entirely on our token system; we only hit Google once (at login).

---

## 📐 Architecture Diagram (ASCII)

```
Client (Browser/Mobile)
    │
    ├── POST /auth/login ─────────────────────────────► Auth Service
    │         Response: accessToken (JWT) + refreshToken (cookie)  │
    │                                                               ├─ PostgreSQL (users, refresh_tokens)
    │                                                               ├─ Redis (revocation blocklist)
    │                                                               └─ Vault (RSA private key)
    │
    ├── GET /api/orders (Authorization: Bearer <accessToken>)
    │         ▼
    │    API Gateway
    │         ├─ Verify JWT signature using public key (from JWKS cache)
    │         ├─ Check revocation blocklist (Redis)
    │         ├─ Inject X-User-Id, X-User-Roles headers
    │         └─ Route to Order Service (no auth logic here)
    │
    └── POST /auth/refresh (refreshToken cookie)
              ▼
         Auth Service → SHA-256(token) → lookup in DB → rotate → new pair
```

---

## 🔗 Related Notes
- [[Asymmetric-JWT-RS256-Implementation]]
- [[Refresh-Tokens-And-Session-Management]]
- [[Spring-Security-Architecture-Filters]]
- [[Method-Level-Security-PreAuthorize]]
- [[API-Gateway-Spring-Cloud-Gateway]]
