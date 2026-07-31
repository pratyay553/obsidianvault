# 09 — API Security

A reference guide covering the core topics in API security: threat landscape, authentication/authorization mechanics, and defensive controls.

---

## 1. OWASP API Top 10

OWASP maintains a dedicated Top 10 for APIs (distinct from the general Web Top 10), because APIs fail in different ways than server-rendered apps. Current categories:

1. **API1: Broken Object Level Authorization (BOLA/IDOR)** — a user can access another user's object just by changing an ID (`/orders/1234` → `/orders/1235`) because the server never checks ownership.
2. **API2: Broken Authentication** — weak token issuance, no rate limiting on login, predictable API keys, missing token expiry.
3. **API3: Broken Object Property Level Authorization** — over-fetching data or allowing mass-assignment of fields a user shouldn't be able to read/write (e.g. setting `"isAdmin": true` in a signup payload).
4. **API4: Unrestricted Resource Consumption** — no limits on payload size, pagination, or request rate, enabling DoS or cost-based attacks.
5. **API5: Broken Function Level Authorization (BFLA)** — a regular user can call admin-only endpoints because the endpoint doesn't check role, only that a token is valid.
6. **API6: Unrestricted Access to Sensitive Business Flows** — automatable abuse of legitimate flows (bulk-buying limited stock, mass account creation) without bot/behavioral protection.
7. **API7: Server-Side Request Forgery (SSRF)** — API fetches a URL supplied by the client (webhooks, image-from-URL) and an attacker points it at internal infrastructure.
8. **API8: Security Misconfiguration** — verbose errors, default credentials, permissive CORS, missing security headers, unpatched dependencies.
9. **API9: Improper Inventory Management** — old/undocumented/shadow API versions still live and unmonitored (e.g. `/v1/` deprecated but reachable, no auth updates applied).
10. **API10: Unsafe Consumption of APIs** — trusting data/responses from third-party APIs without validation, as if they were internal and safe.

**Practical takeaway:** BOLA and BFLA (1 and 5) are consistently the most common real-world API breaches — they're authorization bugs, not authentication bugs, and scanners rarely catch them. Manual/business-logic testing is required.

---

## 2. JWT Security

JSON Web Tokens are a common bearer-token format (`header.payload.signature`), but they're widely misused.

**Common failures:**

- **`alg: none` acceptance** — some libraries historically accepted unsigned tokens if the header said so. Always enforce an explicit allow-list of algorithms server-side.
- **Algorithm confusion (RS256 → HS256)** — if a server is configured to accept both, an attacker can take the public RSA key (often published) and use it as the HMAC secret to forge a validly-signed token.
- **Weak/leaked signing secrets** — short or guessable HMAC secrets can be brute-forced offline.
- **No expiry or excessive expiry (`exp`)** — long-lived JWTs can't be revoked; a stolen token stays valid indefinitely.
- **Sensitive data in the payload** — JWT payloads are base64-encoded, not encrypted. Never put secrets or PII in the claims.
- **No revocation mechanism** — JWTs are stateless by design, so logout/ban doesn't invalidate existing tokens unless you maintain a deny-list or use short expiry + refresh tokens.

**Best practices:**

- Enforce the algorithm server-side (don't trust the `alg` header claim).
- Use short access-token lifetimes (minutes) with a separate refresh token flow.
- Validate `iss`, `aud`, `exp`, `nbf` on every request.
- Store secrets/keys in a secrets manager, rotate periodically.
- Consider opaque reference tokens (validated against a session store) for high-sensitivity APIs instead of JWTs, if revocation matters more than statelessness.

---

## 3. API Gateway

An API gateway is the single entry point that sits in front of backend services and centralizes cross-cutting concerns:

- **AuthN/AuthZ enforcement** — validate tokens/API keys before traffic reaches services.
- **Rate limiting & throttling** — per-client, per-key, per-endpoint quotas.
- **Request/response transformation** — schema validation, header injection, protocol translation.
- **TLS termination** — centralizes certificate management.
- **Routing & versioning** — directs `/v1/`, `/v2/` traffic, can retire deprecated versions cleanly (mitigates OWASP API9).
- **Observability** — centralized logging, metrics, and anomaly detection across all APIs.

**Security value:** a gateway reduces the chance that any individual microservice reinvents (and gets wrong) authentication or rate limiting. But it's a single point of failure/compromise — gateway misconfiguration (e.g. permissive CORS, missing auth on internal routes) exposes everything behind it.

---

## 4. mTLS (Mutual TLS)

Standard TLS authenticates the server to the client. **Mutual TLS** authenticates both directions: the client also presents a certificate, which the server validates against a trusted CA.

**Use cases:**

- Service-to-service communication inside a microservices mesh (e.g. via Istio/Linkerd sidecars).
- B2B API integrations where both parties are known organizations, not anonymous end users.
- Zero-trust network architectures, where network location no longer implies trust.

**Mechanics:**

1. Client and server both present X.509 certificates during the TLS handshake.
2. Each side validates the other's cert against a trusted CA and checks revocation (CRL/OCSP).
3. Once established, the connection is encrypted and both identities are cryptographically verified.

**Trade-offs:** strong identity guarantees, but adds operational overhead — certificate issuance, rotation, and revocation at scale. Not typically used for public-facing consumer APIs (you can't distribute private keys to anonymous browsers); more common for internal/partner APIs.

---

## 5. OAuth2 (for APIs)

OAuth2 is an **authorization** framework (not authentication — that's OpenID Connect layered on top). It lets a client obtain limited, delegated access to a resource on behalf of a user without handling the user's credentials directly.

**Key grant types relevant to APIs:**

- **Authorization Code (+ PKCE)** — standard for web/mobile apps with a user present. PKCE is now mandatory best practice even for confidential clients, to prevent code interception attacks.
- **Client Credentials** — machine-to-machine, no user context; a service authenticates as itself.
- **Refresh Token** — used to obtain new access tokens without re-prompting the user; should be rotated on each use (refresh token rotation) to detect theft.
- **Deprecated/discouraged:** Implicit grant and Resource Owner Password Credentials — both considered insecure and removed from OAuth 2.1.

**API security considerations:**

- Always validate the `aud` (audience) claim so a token issued for API A can't be replayed against API B.
- Bind tokens to scopes (see below) — never issue a token with blanket access.
- Use short-lived access tokens + refresh token rotation instead of long-lived static tokens.
- Validate redirect URIs strictly (exact match, not prefix match) to prevent authorization code interception.

---

## 6. Rate Limiting (API)

Mitigates OWASP API4 (unrestricted resource consumption), brute-force/credential-stuffing attacks, and scraping.

**Common strategies:**

- **Fixed window** — simple counters per time window; can allow bursts at window boundaries.
- **Sliding window / sliding log** — more accurate, smooths out boundary bursts.
- **Token bucket / leaky bucket** — allows controlled bursts while enforcing an average rate; widely used (e.g. AWS API Gateway, NGINX).

**Dimensions to limit on:**

- Per API key / per user (not just per IP — IPs are shared behind NAT/CGNAT and easily rotated).
- Per endpoint (expensive endpoints like search/export need tighter limits than a health check).
- Global limits to protect backend capacity regardless of per-client fairness.

**Response behavior:** return `429 Too Many Requests` with a `Retry-After` header. Avoid leaking exactly how close a client is to the limit in ways that help attackers tune requests to just under the threshold, though standard `X-RateLimit-*` headers are generally fine for legitimate integrators.

---

## 7. API Keys

A simple, static credential (long random string) sent with each request, typically for identifying an application/client rather than a human user.

**Weaknesses:**

- No inherent expiry — a leaked key (committed to a public repo, embedded in a mobile app) is valid until manually revoked.
- No scoping by default unless explicitly designed — often carries broad access.
- Frequently transmitted in query strings, which get logged by proxies, browser history, and referrer headers — should always be sent in headers instead.

**Best practices:**

- Treat as secrets: never embed in client-side/mobile code where they can be extracted.
- Scope keys narrowly (read-only vs write, specific resources).
- Rotate regularly and support multiple active keys per client to allow zero-downtime rotation.
- Log key usage and alert on anomalous patterns (sudden volume spike, new geography).
- For anything user-specific or sensitive, prefer OAuth2 tokens over static API keys — keys authenticate an _app_, not a _user_.

---

## 8. Scopes

Scopes implement the principle of least privilege within a token — they declare **what** a token is allowed to do, separate from **who** it belongs to.

Example: an OAuth2 access token might carry `scope: "orders:read profile:read"`, meaning it can read orders and profile data but cannot write, delete, or access billing.

**Design guidance:**

- Scope granularly by resource and action (`orders:read`, `orders:write`) rather than coarse buckets (`full_access`).
- Enforce scope checks at the API layer on every request — issuing a scoped token is meaningless if the backend doesn't actually check it (this is what makes API3/API5 in the OWASP list possible).
- Distinguish scopes from roles: scopes limit what a _token_ can do; roles/permissions determine what a _user_ can do. A token's effective access should be the intersection of both.
- Display requested scopes to users during consent (OAuth authorization screens) so delegation is informed, not blind.

---

## 9. Pagination Abuse

Pagination parameters, if trusted blindly, become an attack surface:

- **Resource exhaustion** — a client requesting `?limit=1000000` or a deeply nested `?offset=50000000` can force expensive database scans. Always enforce a server-side max page size regardless of what the client requests.
- **Enumeration / scraping** — sequential IDs combined with unrestricted pagination let an attacker walk an entire dataset (e.g. every user profile) quickly. Mitigate with rate limiting, opaque cursor-based pagination instead of offset-based, and authorization checks per record (not just "is this request allowed" but "is _this specific item_ allowed for _this_ caller).
- **Cursor tampering** — if cursors are just base64-encoded offsets or IDs, attackers can forge them to skip authorization boundaries. Sign or encrypt opaque cursors server-side.
- **Inconsistent authorization under pagination** — a classic BOLA variant: list endpoints sometimes forget to apply the same per-object ownership check that detail endpoints enforce, because "it's just a list."

**Best practice:** prefer cursor-based (keyset) pagination over offset-based for both performance and security, cap page sizes server-side, and apply object-level authorization on every item returned, not just at the collection level.

---

## 10. Input Validation (API)

APIs receive structured input (JSON/XML/form-data/protobuf) but still need the same rigor as any other user input.

**Key controls:**

- Validate **type, format, range, and length** for every field — reject unknown/unexpected fields rather than silently ignoring them (prevents mass-assignment attacks, API3).
- Validate at the API boundary, not just in business logic deeper in the stack — fail fast with a clear `400` rather than letting malformed data propagate.
- Canonicalize and validate encodings (UTF-8, no null bytes, no NoSQL/SQL injection payloads in string fields).
- Set explicit size limits — request body size, array lengths, string lengths, nesting depth (deeply nested JSON can cause parser DoS).
- Never rely solely on client-side validation; the API must independently enforce every rule, since clients are fully attacker-controlled.
- Sanitize/validate file uploads separately: content-type sniffing, size limits, and storage outside the web root.

---

## 11. Schema Validation

A stricter, structural form of input validation: requests (and often responses) are validated against a formal schema — **JSON Schema**, **OpenAPI/Swagger definitions**, or **Protocol Buffers** for gRPC.

**Why it matters for security:**

- Enforces a strict contract: unexpected fields, wrong types, or malformed structure are rejected before touching business logic — a strong defense against mass assignment and injection payloads riding in unexpected fields.
- Can be automated: generate validation middleware directly from the OpenAPI spec so the spec _is_ the enforcement layer, not just documentation that drifts out of sync with reality.
- Helps catch **API9 (improper inventory management)** — a maintained, validated schema per version makes it obvious which endpoints/fields are actually live versus stale documentation.

**Practical setup:** validate incoming requests against the OpenAPI schema at the gateway or middleware layer, reject anything non-conformant with a `400` before it reaches application code, and keep the schema itself in version control and CI so drift is caught early.

---

## 12. Replay Protection

Prevents a captured, valid request from being resent later (or by someone else) to fraudulent effect — relevant even over TLS, since TLS protects transport confidentiality but not application-layer replay.

**Common techniques:**

- **Nonces** — a unique, single-use value per request; the server tracks recently-seen nonces and rejects duplicates.
- **Timestamps + short validity window** — reject requests whose timestamp is too far from server time (e.g. ±5 minutes), limiting the replay window.
- **HMAC request signing** — client signs the request (body + timestamp + nonce) with a shared secret; server recomputes and compares. Common in webhook verification (e.g. Stripe, GitHub webhooks) and B2B API integrations.
- **Sequence numbers** — monotonically increasing per-client counters; server rejects anything not greater than the last seen value (used in some financial/messaging protocols).

**Where it matters most:** webhook receivers, payment/financial APIs, and any state-changing (non-idempotent) endpoint where a captured request replayed later could cause duplicate effects (double charge, duplicate order).

---

### Quick cross-reference

|Control|Primarily defends against|
|---|---|
|OWASP API Top 10|Overall threat taxonomy|
|JWT security|Token forgery, broken authentication|
|API Gateway|Centralizes all controls below|
|mTLS|Service identity spoofing (internal/partner APIs)|
|OAuth2|Delegated access without credential sharing|
|Rate limiting|Resource exhaustion, brute force|
|API keys|Client/app identification|
|Scopes|Least-privilege enforcement|
|Pagination controls|Data scraping, enumeration, DoS|
|Input validation|Injection, mass assignment, malformed data|
|Schema validation|Structural contract enforcement|
|Replay protection|Request/token reuse, duplicate transactions|