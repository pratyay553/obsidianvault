# 09 — API Security

This is a simple guide to API security. It covers 12 important topics. Each section explains the idea in plain language, with examples.

---

## 1. OWASP API Top 10

OWASP is an organization that tracks common security problems. They made a special "Top 10" list just for APIs, because APIs break in different ways than normal websites.

Here are the 10 problems:

1. **Broken Object Level Authorization (BOLA)** — A user changes an ID in the URL, like `/orders/1234` to `/orders/1235`, and sees someone else's data. This happens because the server does not check who owns the object.
2. **Broken Authentication** — Login and tokens are weak. Example: no limit on login attempts, or API keys that are easy to guess.
3. **Broken Object Property Level Authorization** — The API sends back more data than needed, or lets a user change fields they should not touch. Example: a user sends `"isAdmin": true` during signup and it works.
4. **Unrestricted Resource Consumption** — There is no limit on request size, request speed, or how much data one call can pull. This can crash the server or cost a lot of money.
5. **Broken Function Level Authorization (BFLA)** — A normal user can call an admin-only action, because the server only checks "is this a valid token," not "does this user have permission."
6. **Unrestricted Access to Sensitive Business Flows** — Attackers automate a normal feature to abuse it. Example: buying all limited-stock items using a bot, again and again.
7. **Server-Side Request Forgery (SSRF)** — The API fetches a URL given by the user (like an image link), and an attacker uses this to reach internal systems that should be private.
8. **Security Misconfiguration** — Simple mistakes like showing detailed error messages, using default passwords, or allowing any website to call the API (bad CORS settings).
9. **Improper Inventory Management** — Old API versions are still running and forgotten. Nobody is watching them, so they become an easy target.
10. **Unsafe Consumption of APIs** — The app trusts data coming from another company's API without checking it, as if it were always safe.

**Simple takeaway:** Problems 1 and 5 (BOLA and BFLA) are the most common in real attacks. These are about permissions, not passwords. Automated scanners usually cannot find these bugs — a person needs to test the logic by hand.

---

## 2. JWT Security

A JWT (JSON Web Token) is a common way to send login information with each request. It has three parts: `header.payload.signature`.

**Common mistakes:**

- **Accepting `alg: none`** — Some old code accepted a token with no signature, if the token itself said "no signature needed." The server should never trust this; it should decide the allowed signing method itself.
- **Algorithm mix-up (RS256 to HS256)** — If a server accepts both signing methods, an attacker can take the public key (often not secret) and use it to fake a valid signature.
- **Weak secret key** — A short or simple secret can be guessed by a computer trying many options.
- **No expiry time** — If a token never expires, a stolen token works forever.
- **Secret data inside the token** — The token's payload is not encrypted, only encoded. Anyone can read it. Never put passwords or private data inside.
- **No way to cancel a token** — JWTs work without checking a database, so logging out does not actually stop the token from working, unless you build extra logic for this.

**Good practices:**

- Decide the signing method on the server. Do not trust what the token claims.
- Make tokens expire quickly (a few minutes), and use a separate "refresh token" to get new ones.
- Always check the fields `iss`, `aud`, `exp`, and `nbf` on every request.
- Keep secret keys in a secure vault, and change them from time to time.
- For very sensitive systems, consider using a simple random token instead of a JWT, and check it against a database. This makes it easy to cancel.

---

## 3. API Gateway

An API gateway is like a front door for all your APIs. Every request passes through it first. It handles common jobs so each service does not repeat the same code:

- **Checking login and permissions** before the request goes further.
- **Limiting how many requests** each user or app can send.
- **Checking and changing requests** — for example, validating the format, or adding headers.
- **Handling encryption (TLS)** in one place.
- **Directing traffic** to the right version, like `/v1/` or `/v2/`, and safely turning off old versions.
- **Watching everything** — logs, metrics, and alerts for unusual activity, all in one place.

**Why it helps security:** Without a gateway, each small service might build its own login and rate-limit code, and some will get it wrong. A gateway reduces mistakes.

**Risk:** The gateway is a single, important target. If it is set up wrong (like allowing any website to call it), everything behind it is exposed.

---

## 4. mTLS (Mutual TLS)

Normal TLS (the padlock in your browser) only proves the server's identity to you. **Mutual TLS** also proves your identity to the server. Both sides show a certificate.

**When it is used:**

- Communication between internal services (microservices).
- APIs shared between two companies who already know and trust each other.
- "Zero trust" systems, where nothing is trusted just because it is on the same network.

**How it works, step by step:**

1. Both the client and the server present a certificate during the connection setup.
2. Each side checks the other's certificate against a trusted authority, and confirms it has not been cancelled.
3. Once both sides are confirmed, the connection is encrypted and both identities are known.

**Trade-off:** mTLS gives strong proof of identity, but it takes extra work to create, renew, and cancel certificates. It is not common for public apps used by everyday people, since you cannot safely give a private certificate to every phone or browser. It works well for internal or partner systems.

---

## 5. OAuth2 (for APIs)

OAuth2 is a system for giving limited access to an app, without sharing your password with that app. (Note: OAuth2 is about _permission_, not about proving who you are — that part is handled by a related standard called OpenID Connect.)

**Main flows used with APIs:**

- **Authorization Code + PKCE** — The standard choice for websites and mobile apps used by a real person. PKCE adds extra protection and should always be used now.
- **Client Credentials** — Used when one system talks to another system directly, with no human user involved.
- **Refresh Token** — Used to get a new access token without asking the user to log in again. Each time it is used, a new refresh token should be issued, so a stolen one can be noticed.
- **Avoid these:** the old "Implicit" flow and the "Password" flow are no longer considered safe.

**Extra tips for APIs:**

- Always check the `aud` (audience) field, so a token made for API A cannot be reused on API B.
- Give each token only the permissions it needs (see Scopes, below).
- Use short-lived tokens with refresh tokens, instead of one token that lasts forever.
- Check redirect URLs exactly. Do not allow "close enough" matches, or attackers can steal the login code.

---

## 6. Rate Limiting (API)

Rate limiting stops one user from sending too many requests. It protects against server overload, password-guessing attacks, and data scraping.

**Common methods:**

- **Fixed window** — Count requests in fixed time blocks (like every minute). Simple, but allows a burst right at the edge of two windows.
- **Sliding window** — A more accurate version of the above; it smooths out that edge problem.
- **Token bucket** — Allows some burst of requests, but keeps the average rate steady over time. This is a very common choice.

**What to limit by:**

- Per user or per API key — not just per IP address, since many users can share one IP address.
- Per endpoint — an expensive action (like search or export) needs a tighter limit than a simple health check.
- Overall system limit — to protect the server no matter who is asking.

**What to send back:** Use the response code `429 Too Many Requests`, along with a `Retry-After` header telling the client when to try again.

---

## 7. API Keys

An API key is a simple, fixed secret string sent with each request. It usually identifies an app, not a specific person.

**Weaknesses:**

- It usually does not expire. If it leaks (for example, in public code on GitHub), it stays valid until someone manually cancels it.
- It often has full access, unless the developer specifically limits it.
- It is sometimes sent in the URL, which gets saved in logs and browser history. It should always be sent in a header instead.

**Good practices:**

- Treat it like a password. Never put it inside mobile app code or front-end JavaScript, where anyone can extract it.
- Give each key only the access it needs.
- Change keys regularly, and allow more than one active key per app, so you can switch keys without downtime.
- Watch how each key is used, and get an alert if usage suddenly looks strange.
- For anything tied to a specific person, use OAuth2 tokens instead of a plain API key. A key proves which _app_ is calling, not which _person_.

---

## 8. Scopes

A scope tells the system exactly **what** a token is allowed to do. This is different from **who** owns the token.

Example: A token might have `scope: "orders:read profile:read"`. This means it can read orders and read the profile, but it cannot write, delete, or see billing information.

**How to design scopes well:**

- Make scopes specific, like `orders:read` and `orders:write`, instead of one big scope like `full_access`.
- Always check the scope on the server, on every request. A scoped token is useless if the server never checks it — this is exactly how problems 3 and 5 from the OWASP list happen.
- Remember: scopes limit what the _token_ can do. Roles limit what the _user_ can do. The real access should be whichever is smaller.
- When a user is asked to give permission (during OAuth login), show them clearly what scopes they are approving.

---

## 9. Pagination Abuse

Pagination means splitting a long list into pages. If the API trusts the page settings blindly, this becomes a security risk:

- **Overload** — If a user asks for `?limit=1000000`, the server may try to load too much data at once. Always set a maximum page size on the server, no matter what the user asks for.
- **Data scraping** — If IDs are simple numbers and pages have no limit, an attacker can quickly download an entire database, page by page. Fix this with rate limiting, and by checking permission for each item, not just for the list as a whole.
- **Fake page tokens** — If the "next page" token is just a plain number or ID, an attacker can change it to skip past permission checks. Page tokens should be signed or encrypted by the server.
- **Missing permission checks in lists** — A common mistake: the single-item page checks "does this user own this item," but the list page forgets to check the same thing for each item.

**Good practice:** Use cursor-based pagination (a safe token for "next page") instead of plain offsets. Always set a max page size. Always check permission for every item returned, not just the list itself.

---

## 10. Input Validation (API)

APIs usually receive structured data, like JSON. This data still needs careful checking, just like any form on a website.

**Key rules:**

- Check the **type, format, length, and range** of every field. Reject any field you do not expect, instead of quietly ignoring it. This stops attackers from sneaking in extra fields, like `"isAdmin": true`.
- Check the data at the very first entry point, not deep inside the business logic. Fail early with a clear `400` error.
- Check for bad characters and dangerous patterns, like SQL code hidden inside a text field.
- Set clear limits: maximum request size, maximum array length, maximum text length, and maximum nesting depth (a JSON with too many nested layers can crash a parser).
- Never trust checks done only in the browser or app. The API itself must check everything again, since a user can bypass the app and call the API directly.
- Check file uploads separately: confirm the real file type, limit the size, and store files outside the main web folder.

---

## 11. Schema Validation

Schema validation is a stricter, more formal version of input checking. The request (and sometimes the response) is compared against a fixed structure, called a schema. Common formats are **JSON Schema**, **OpenAPI**, or **Protocol Buffers**.

**Why this matters for security:**

- It creates a strict contract. Any unexpected field, wrong type, or bad structure is rejected before it reaches the actual code. This is a strong defense against the "extra field" attacks mentioned above.
- It can be automated. You can generate the checking code directly from your OpenAPI file, so the documentation and the actual security check are always the same thing.
- It helps you spot old, forgotten API versions (problem 9 from the OWASP list), since a well-kept schema makes it obvious which endpoints are really active.

**How to set it up:** Check every incoming request against your OpenAPI schema, at the gateway or in early middleware. Reject anything that does not match, with a `400` error. Keep the schema file in your code repository, and test it automatically, so it never becomes outdated.

---

## 12. Replay Protection

Replay protection stops an old, valid request from being captured and sent again later, either by an attacker or by accident. This matters even with HTTPS, because HTTPS protects the data in transit, but not against someone resending an already-sent message.

**Common methods:**

- **Nonce** — A random, one-time value included in each request. The server remembers recent nonces and rejects any repeat.
- **Timestamp with a short time window** — The server rejects any request with a timestamp too far from the current time, like more than 5 minutes old.
- **Signed requests (HMAC)** — The client signs the request (data plus timestamp plus nonce) using a shared secret. The server re-checks the signature. This is common for webhooks, like those from Stripe or GitHub.
- **Sequence numbers** — Each client sends requests with an always-increasing number. The server rejects any number that is not higher than the last one it saw. Used often in banking and messaging systems.

**Where this matters most:** Webhooks, payment APIs, and any action that changes data (not just reads it), where resending a request could cause a real problem, like charging someone twice.

---

### Quick summary table

|Topic|Main problem it stops|
|---|---|
|OWASP API Top 10|A full list of common API risks|
|JWT security|Fake or stolen login tokens|
|API Gateway|Central place for all the checks below|
|mTLS|Fake service identity (internal/partner APIs)|
|OAuth2|Sharing access without sharing passwords|
|Rate limiting|Server overload, password guessing|
|API keys|Identifying which app is calling|
|Scopes|Giving only the access that is needed|
|Pagination controls|Data scraping, overload|
|Input validation|Bad or malicious data getting in|
|Schema validation|Requests that break the expected format|
|Replay protection|Old requests or tokens being reused|
