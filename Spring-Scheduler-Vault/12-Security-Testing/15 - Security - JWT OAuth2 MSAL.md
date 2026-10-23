---
tags: [security, jwt, oauth2, interview]
---

# Security — JWT, OAuth2, MSAL4J

Related: [[11 - Email Services]] · [[07 - REST and SOAP Clients]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
A backend scheduler still needs to **authenticate itself** to the external systems and APIs it talks to — there's no human logging in, so it uses machine-to-machine auth patterns.

## 🔑 Core Concepts

| Concept | What it is |
|---|---|
| **JWT** (JSON Web Token) | A compact, signed token containing claims (who, what permissions, expiry) — verifiable without a DB lookup because the signature proves it wasn't tampered with |
| **OAuth2** | The *protocol/framework* for obtaining access tokens (JWTs are often the token format used) |
| **MSAL4J** | Microsoft's library implementing OAuth2 flows specifically for Azure AD / Microsoft 365 |
| `jjwt` | A popular Java library for **creating and parsing** JWTs directly (when you're not going through a full OAuth2 provider) |

## 🛠️ JWT Structure (know this cold for interviews)
```
header.payload.signature
```
```json
// header
{ "alg": "HS256", "typ": "JWT" }
// payload (claims)
{ "sub": "scheduler-service", "exp": 1735689600, "scope": "read:trips" }
```
The signature is computed over `base64(header) + "." + base64(payload)` using a secret (HMAC) or private key (RSA/EC) — anyone can *read* a JWT's payload (it's just base64, not encrypted), but only the issuer can *produce a valid signature*.

## 🛠️ Client Credentials Flow (the one machine-to-machine schedulers use)

```java
ConfidentialClientApplication app = ConfidentialClientApplication.builder(
        clientId, ClientCredentialFactory.createFromSecret(clientSecret))
        .authority("https://login.microsoftonline.com/" + tenantId)
        .build();

CompletableFuture<IAuthenticationResult> future = app.acquireToken(
        ClientCredentialParameters.builder(Set.of("https://graph.microsoft.com/.default")).build());

String accessToken = future.get().accessToken();
```
This is **OAuth2's Client Credentials grant** — no user is involved; the *application itself* is the identity, authenticating with a `clientId` + `clientSecret` (or certificate) to get a token it then attaches as `Authorization: Bearer <token>` on outbound calls.

## ❓ Interview Q&A

**Q: Why is JWT considered "stateless" authentication?**
A: The server doesn't need to store session data — all the necessary claims (identity, permissions, expiry) travel inside the signed token itself. Any service with the public key/shared secret can verify it independently, without a shared session store.

**Q: Is a JWT encrypted?**
A: No — by default a JWT is only **signed**, not encrypted; the payload is base64-encoded (trivially decodable), not confidential. If you need to hide the contents, you'd use JWE (JSON Web Encryption) or simply avoid putting sensitive data in the payload.

**Q: Which OAuth2 grant type does a backend scheduler use, and why not Authorization Code?**
A: **Client Credentials grant** — because there's no human user to redirect through a login screen. Authorization Code grant is for user-facing apps where a person logs in and consents; a scheduled backend job authenticates as itself.

**Q: How do you handle token expiry in a long-running scheduled service?**
A: Cache the access token along with its expiry, and refresh proactively (e.g., request a new token when the cached one has, say, less than 5 minutes left) rather than acquiring a brand-new token on every single call — MSAL4J's `ConfidentialClientApplication` actually does in-memory token caching for you by default.
