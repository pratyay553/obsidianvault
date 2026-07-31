---

## title: "05 - Authentication" tags: [cybersecurity, appsec, authentication, interview-prep, web-security] created: 2026-07-16

# 05 - Authentication

> **Quick context before we start, bhai:** "Authentication" = proving **WHO YOU ARE** (login). Don't confuse this with **Authorization** = what you're ALLOWED to do (permissions), that's a separate topic. Interviewers LOVE asking "what's the difference between authentication and authorization" as a warm-up question — one line answer: **"Authentication is 'who are you', Authorization is 'what can you do'."**
> 
> Each topic below follows the same structure:
> 
> - 🔹 **What it is**
> - 🔹 **How it works** (step by step flow)
> - 🔹 **Real Example**
> - 🔹 **Security Issues / Attacks** (interviewer's favorite follow-up)
> - 🔹 **💬 Interview One-Liner**

---

## Table of Contents

1. [[#1. OAuth2]]
2. [[#2. OpenID Connect (OIDC)]]
3. [[#3. JWT (JSON Web Token)]]
4. [[#4. Session Authentication]]
5. [[#5. MFA (Multi-Factor Authentication)]]
6. [[#6. Passkeys]]
7. [[#7. SAML]]
8. [[#8. Kerberos]]
9. [[#9. LDAP]]
10. [[#Comparison Table - Which One Is Used Where]]
11. [[#Quick Revision Table (Night Before Interview)]]

---

## 1. OAuth2

**What it is:** OAuth2 is an **authorization framework** (not authentication!) that lets a third-party app get **limited access** to a user's resources on another service, WITHOUT ever seeing the user's password.

> ⚠️ **Interview trap:** OAuth2 is NOT for login/authentication by itself — it's about **delegated access**. People say "Login with Google" and think that's OAuth2 authenticating them, but actually that's OpenID Connect (built ON TOP of OAuth2) doing the authentication part. OAuth2 alone just answers "can this app access my Google Drive files?"

**How it works (Authorization Code flow — most common & most asked in interview):**

1. User clicks "Connect to Google Drive" on ThirdPartyApp
2. ThirdPartyApp redirects user to Google's authorization server with `client_id`, `redirect_uri`, `scope`, `state`
3. User logs into Google (if not already) and sees consent screen: "ThirdPartyApp wants to access your Drive files"
4. User clicks "Allow" → Google redirects back to ThirdPartyApp's `redirect_uri` with a temporary **authorization code**
5. ThirdPartyApp's backend exchanges this code + its `client_secret` for an **access token** (server-to-server call, code never exposed to browser after this)
6. ThirdPartyApp uses the access token to call Google Drive API on user's behalf

**Key Roles/Terms to remember:**

- **Resource Owner** = the user
- **Client** = the third-party app (ThirdPartyApp)
- **Authorization Server** = issues tokens (Google's auth server)
- **Resource Server** = holds the actual data (Google Drive API)
- **Scope** = what permissions are being requested (`read-only`, `drive.files`)
- **Access Token** = short-lived token to call APIs
- **Refresh Token** = long-lived token to get new access tokens without re-login

**Grant Types (flows) to remember:**

- **Authorization Code** – most secure, used by web/mobile apps (with PKCE now standard)
- **Client Credentials** – machine-to-machine, no user involved (service talks to service)
- **PKCE (Proof Key for Code Exchange)** – extension for public clients (mobile/SPA apps) that can't safely store a `client_secret`
- ~~Implicit~~ and ~~Password~~ grants — **deprecated/discouraged now**, don't recommend these in interview

**Real Example:** "Login with Google" on a random blog site, or a travel app asking "Allow access to your Google Calendar to add trip dates" — that consent screen you see is OAuth2 in action.

**Security Issues / Attacks:**

- **`state` parameter missing** → CSRF attack on the OAuth flow (attacker links their own authorization code to victim's session)
- **Open redirect via `redirect_uri`** if not strictly validated (allow-list exact URIs, don't just check domain)
- **Authorization code interception** on public clients (mobile apps) → fixed by **PKCE**
- **Using Implicit flow** → access token exposed in URL fragment, easily leaked via browser history/Referer

**Prevention:**

- Always validate `redirect_uri` against an exact allow-list
- Always use `state` parameter to prevent CSRF
- Use PKCE for all public clients (mobile, SPA)
- Never use Implicit or Password grant in new implementations
- Short-lived access tokens + refresh token rotation

**💬 Interview One-Liner:** "OAuth2 is an authorization framework for delegated access — it lets an app access a user's data on another service without ever seeing their password, using the Authorization Code flow with PKCE as the modern standard."

---

## 2. OpenID Connect (OIDC)

**What it is:** OIDC is an **authentication layer built on top of OAuth2**. It answers "who is this user" (identity), while OAuth2 alone only answers "what can this app access". This is THE actual technology behind "Sign in with Google/Facebook/Microsoft" buttons.

**How it works:**

1. Same flow as OAuth2 Authorization Code flow above, BUT the app also requests the special `openid` scope
2. Along with the access token, the authorization server ALSO returns an **ID Token** — this is a **JWT** containing the user's identity info (`sub` = user ID, `email`, `name`, `iss` = issuer, `aud` = audience, `exp` = expiry)
3. The app verifies the ID Token's signature (using the issuer's public key from a `.well-known/jwks.json` endpoint) to confirm the user's identity
4. App now knows exactly WHO logged in (not just "has permission to X resource")

**Real Example:** "Sign in with Google" button — after you approve, the app receives an ID Token (JWT) saying `{"sub": "1234567890", "email": "you@gmail.com", "email_verified": true, "name": "Your Name"}` — this is how the app knows your identity without you ever creating a password on that site.

**Key difference to always mention in interview:**

|OAuth2|OpenID Connect|
|---|---|
|Authorization ("what can you access")|Authentication ("who are you")|
|Returns **Access Token**|Returns **Access Token + ID Token (JWT)**|
|No standard user info format|Standard `userinfo` endpoint + standard claims|

**Security Issues / Attacks:**

- Not validating ID Token signature/issuer/audience properly → attacker can forge identity
- Confusing access token with ID token (access token is NOT meant to carry identity info, ID token is)
- Missing `nonce` parameter → replay attacks on the ID token

**Prevention:**

- Always verify ID Token signature against issuer's public keys (JWKS)
- Validate `iss`, `aud`, `exp`, and `nonce` claims
- Use official, well-tested OIDC client libraries — never write your own token validation

**💬 Interview One-Liner:** "OpenID Connect adds an identity layer on top of OAuth2 — the key giveaway is the ID Token, which is a JWT containing who the user actually is, that's the real magic behind 'Sign in with Google'."

---

## 3. JWT (JSON Web Token)

**What it is:** A compact, self-contained, URL-safe token format used to represent claims (data) between two parties — most commonly used for stateless authentication (server doesn't need to store session data, everything needed is inside the token itself).

**How it works — Structure (3 parts separated by dots):**

```
header.payload.signature
```

- **Header:** algorithm + token type, e.g. `{"alg":"HS256","typ":"JWT"}`
- **Payload:** the actual claims, e.g. `{"sub":"123","role":"admin","exp":1735689600}`
- **Signature:** `HMACSHA256(base64(header) + "." + base64(payload), secret)` — this is what makes it tamper-proof

> ⚠️ **Important interview point:** JWT payload is only **Base64-ENCODED, not encrypted!** Anyone can decode and read it (try it on jwt.io). Never put sensitive data (passwords, secrets) inside a JWT payload — the signature only guarantees it hasn't been **tampered with**, not that it's **secret**.

**Flow in a typical app:**

1. User logs in with username/password
2. Server verifies credentials, creates a JWT, signs it with a secret/private key
3. Server sends JWT to client (usually in response body or cookie)
4. Client sends JWT back on every subsequent request (usually `Authorization: Bearer <token>` header)
5. Server verifies signature on every request — if valid & not expired, trusts the claims inside (no DB lookup needed = "stateless")

**Real Example:** After logging into most modern APIs (banking apps, SaaS dashboards), check dev tools → Application → Local Storage or cookies → you'll usually find a long string with two dots in it — that's your JWT.

**Security Issues / Attacks:** _(detailed versions covered in Application Layer Attacks note, quick recap here)_

- **`alg: none` attack** — removing signature, some libraries wrongly accept it
- **Algorithm confusion (RS256 → HS256)** — using public key as HMAC secret
- **Weak/guessable HMAC secret** → brute-forced offline, attacker forges any token
- **No expiry / long-lived tokens** → stolen token usable forever
- **Storing JWT in localStorage** → vulnerable to theft via XSS (unlike HttpOnly cookies)
- **No revocation mechanism** — since it's stateless, you can't easily "log out" a JWT before its expiry (this is a real architectural weakness, mention it in interview — shows depth!)

**Prevention:**

- Always hardcode/whitelist expected algorithm server-side
- Use strong random secrets (32+ bytes) or RS256 with properly separated keys
- Short expiry (`exp`) + refresh token pattern
- Store JWT in `HttpOnly`, `Secure`, `SameSite` cookie instead of localStorage where possible
- For revocation: maintain a short-lived denylist/blocklist, or keep expiry very short

**💬 Interview One-Liner:** "JWT is a self-contained, signed token — the payload is only Base64-encoded not encrypted, so never put secrets in it, and remember its biggest architectural weakness is that stateless tokens are hard to revoke before they expire."

---

## 4. Session Authentication

**What it is:** The classic, traditional way of maintaining login state — server creates a session and stores session data **server-side**, giving the client only a random session ID (via cookie) to reference it. This is the opposite philosophy of JWT (stateful vs stateless).

**How it works:**

1. User logs in with username/password
2. Server verifies credentials, creates a session object in server-side storage (memory, Redis, DB) containing user info
3. Server generates a random, unguessable **Session ID** and sends it to browser as a cookie (`Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict`)
4. Browser automatically sends this cookie on every subsequent request to that domain
5. Server looks up the session ID in its storage to find out who the user is, on every request
6. Logout = server deletes the session from storage → session ID becomes useless immediately

**Real Example:** Most traditional web apps (Django default auth, PHP `$_SESSION`, old-school banking portals) — you'll see a cookie like `sessionid=` or `JSESSIONID=` or `PHPSESSID=` in dev tools.

**Session vs JWT — the classic interview comparison question:**

|Session Authentication|JWT|
|---|---|
|**Stateful** — server stores session data|**Stateless** — server stores nothing, data is in token|
|Easy to revoke (just delete server-side)|Hard to revoke before expiry|
|Needs shared session store for scaling (Redis) across multiple servers|Scales easily across servers — no shared store needed|
|Session ID is meaningless without server lookup|Token itself contains readable claims (base64)|
|Vulnerable to Session Fixation/Hijacking if IDs leak|Vulnerable to token theft, alg confusion attacks|

**Security Issues / Attacks:**

- **Session Fixation** — attacker fixes a known session ID before login (covered in Application Layer Attacks note)
- **Session Hijacking** — stealing the session cookie (via XSS, network sniffing, etc.)
- **Session ID in URL** — leaks via browser history, proxy logs, Referer headers
- **No session expiry** — stale sessions live forever, larger attack window
- **Predictable session IDs** — if not cryptographically random, attacker can guess valid ones

**Prevention:**

- Regenerate session ID after login (prevents fixation)
- `HttpOnly`, `Secure`, `SameSite=Strict/Lax` cookie flags
- Session expiry (idle timeout + absolute timeout)
- Cryptographically random, long session IDs (128+ bits of entropy)
- Never put session ID in URL

**💬 Interview One-Liner:** "Session Authentication is stateful — the server holds the actual session data and the client just holds a random reference ID — which makes it easy to revoke instantly, unlike JWT, but harder to scale horizontally without a shared session store like Redis."

---

## 5. MFA (Multi-Factor Authentication)

**What it is:** Requiring the user to prove their identity using **two or more independent factors** instead of just a password — massively reduces account takeover risk even if password is leaked/guessed.

**The 3 Classic Factor Categories (memorize this, very commonly asked):**

1. **Something you KNOW** — password, PIN, security question
2. **Something you HAVE** — phone (OTP/SMS), hardware token (YubiKey), authenticator app
3. **Something you ARE** — biometrics (fingerprint, face ID, retina)

> 💡 True MFA needs factors from **different** categories. Asking for password + security question is NOT MFA (both are "something you know")!

**Common MFA Methods:**

- **SMS OTP** — code sent via text (weakest, vulnerable to SIM swapping — mention this weakness in interview to sound sharp!)
- **TOTP (Time-based One-Time Password)** — apps like Google Authenticator/Authy generate a 6-digit code that changes every 30 seconds, based on a shared secret + current time
- **Push Notification** — "Approve this login?" prompt on your phone (Duo, Okta Verify)
- **Hardware Security Keys (FIDO2/U2F)** — physical USB/NFC key like YubiKey, strongest option, phishing-resistant
- **Biometrics** — fingerprint/Face ID, usually combined with device possession

**How TOTP works (good to know the math):**

```
TOTP = HMAC(shared_secret, current_unix_time / 30_seconds) → truncated to 6 digits
```

Both server and authenticator app know the shared secret (set up via QR code scan at enrollment) and independently calculate the same code using current time — no network communication needed at verification time, which is why it works offline!

**Real Example:** Logging into Gmail from a new device → asks for password (factor 1) AND a code from Google Authenticator or a tap on your phone (factor 2).

**Security Issues / Attacks:**

- **SIM Swapping** — attacker convinces telecom to port victim's number to attacker's SIM → receives SMS OTP
- **MFA Fatigue / Push Bombing** — attacker spams push notification approval requests hoping victim accidentally taps "Approve" out of annoyance (this is exactly how the 2022 Uber breach happened!)
- **Phishing (real-time relay/AiTM - Adversary in the Middle)** — attacker's fake login page relays your credentials AND OTP to the real site in real-time, capturing the session
- **Backup codes / account recovery flow** — often weaker than the MFA itself, becomes the actual weak link

**Prevention:**

- Prefer TOTP or hardware keys over SMS wherever possible
- Rate-limit/throttle push notification prompts, require number-matching (not just "Approve/Deny")
- Educate users, monitor for unusual MFA request patterns
- Use FIDO2/WebAuthn (phishing-resistant by design) for high-security accounts

**💬 Interview One-Liner:** "MFA combines factors from different categories — knowledge, possession, inherence — and while SMS OTP is common, it's the weakest due to SIM swapping; hardware keys using FIDO2/WebAuthn are the phishing-resistant gold standard."

---

## 6. Passkeys

**What it is:** A modern, **passwordless** authentication method based on public-key cryptography (built on the **FIDO2/WebAuthn** standard) — no password is ever created, stored, or transmitted. Designed to replace passwords entirely, and to be phishing-resistant by design.

**How it works:**

1. **Registration:** When you set up a passkey for a site, your device (phone/laptop) generates a **key pair** — a private key (stays securely on your device, in secure enclave/TPM) and a public key (sent to and stored by the website)
2. **Login:** Website sends a "challenge" (random data) to your device
3. Your device signs the challenge using the **private key** (after you unlock it via biometric/PIN — this unlocking is LOCAL, never sent over network)
4. Website verifies the signature using the stored **public key** — if valid, you're logged in
5. Since the private key **never leaves your device** and nothing secret is ever transmitted, there's no password to steal, phish, or leak in a breach

**Real Example:** Modern "Sign in with Face ID/Fingerprint" options on Apple/Google/Microsoft accounts, or sites showing "Create a passkey" option instead of "Set a password" — synced across your devices via iCloud Keychain/Google Password Manager.

**Why Passkeys are a big deal (say this in interview to impress!):**

- **Phishing-proof by design** — the private key is cryptographically bound to the exact domain (origin) it was created for. Even if attacker makes a perfect-looking fake site, your device simply won't offer to use the passkey there — no user input can override this.
- **No shared secret ever exists** — unlike passwords (server stores a hash you could theoretically guess/crack) or OTP (a shared secret exists between two parties), the private key never even touches the server.
- **Nothing to leak in a data breach** — server only stores the PUBLIC key, which is useless to an attacker without the matching private key.

**Security Issues / Attacks:**

- **Device loss** — if you lose your only device and haven't synced/backed up the passkey, recovery becomes the account's weak link
- **Platform lock-in concerns** — sync mechanisms differ by ecosystem (Apple/Google/Microsoft), cross-platform passkey portability is still maturing
- **Downgrade attacks** — if a site still offers password as fallback alongside passkeys, attacker can just target the weaker password option instead

**Prevention/Best Practice:**

- Encourage passkey-only accounts where possible (remove password fallback for high-security use cases)
- Ensure proper account recovery flow that doesn't itself become a weak bypass
- Support multiple passkeys per account (backup device)

**💬 Interview One-Liner:** "Passkeys use FIDO2/WebAuthn public-key cryptography — the private key never leaves your device and is cryptographically bound to the exact origin, which makes them phishing-proof by design, unlike passwords or even OTP-based MFA."

---

## 7. SAML

**What it is:** SAML (Security Assertion Markup Language) is an **XML-based** standard for exchanging authentication and authorization data between an **Identity Provider (IdP)** and a **Service Provider (SP)** — the classic backbone of enterprise **Single Sign-On (SSO)**.

**How it works (SP-Initiated flow — most common):**

1. User tries to access `SalesApp.com` (the Service Provider) without being logged in
2. SalesApp redirects user's browser to the company's Identity Provider (e.g., Okta, Azure AD, ADFS)
3. User authenticates at the IdP (enters corporate credentials, maybe with MFA)
4. IdP generates a **SAML Assertion** — a signed XML document stating "this user is verified, here's their identity/attributes" — and sends it back to the browser
5. Browser auto-submits this assertion (usually via an auto-submitting HTML form, POST binding) to SalesApp's Assertion Consumer Service (ACS) URL
6. SalesApp verifies the assertion's digital signature (using IdP's public certificate) and logs the user in — no password ever touched SalesApp!

**Real Example:** Logging into Salesforce, Workday, or any corporate tool via your company's Okta/Azure AD portal — click one tool in the company dashboard, and you're instantly logged in everywhere without re-entering a password. This is enterprise SSO, and it's almost always SAML (or OIDC in newer setups) under the hood.

**Key Terms to remember:**

- **IdP (Identity Provider)** — verifies identity (Okta, Azure AD, ADFS, PingFederate)
- **SP (Service Provider)** — the actual app wanting to authenticate the user (Salesforce, Workday)
- **SAML Assertion** — the signed XML "proof of identity" document
- **ACS URL** — where the SP receives the assertion

**Security Issues / Attacks:**

- **XML Signature Wrapping (XSW)** — attacker manipulates the XML structure to trick the SP into validating the signature on one part of the document while actually reading identity claims from a different, attacker-modified part
- **XXE via SAML** — since SAML is XML-based, poorly configured XML parsers processing SAML assertions can be vulnerable to XXE (covered in Application Layer Attacks note!)
- **Assertion Replay** — capturing and resending a valid assertion if no proper expiry/one-time-use enforcement
- **Missing/weak signature validation** — SP not properly verifying the IdP's signature, accepting forged assertions

**Prevention:**

- Strict, correct XML signature validation (validate the SIGNED portion matches what's being read)
- Disable DTD/external entity processing in the XML parser (same fix as XXE)
- Enforce assertion expiry (`NotOnOrAfter` condition) and one-time use
- Use well-tested SAML libraries, never hand-roll XML signature validation

**💬 Interview One-Liner:** "SAML is XML-based enterprise SSO — an Identity Provider sends a signed XML Assertion to the Service Provider, and because it's XML, it's specifically vulnerable to XXE and signature wrapping attacks if the parser isn't locked down properly."

---

## 8. Kerberos

**What it is:** A network authentication protocol (originally from MIT) that uses **tickets** and **symmetric-key cryptography** to let users prove their identity to multiple services WITHOUT sending passwords over the network — the core authentication mechanism behind Windows Active Directory domains.

**How it works (simplified — 3 main servers/steps, remember this as "AS → TGS → Service"):**

1. **Authentication Server (AS) step:** User logs into their workstation. Client sends a request to the **AS** (part of the KDC - Key Distribution Center). AS verifies user exists, and sends back a **TGT (Ticket Granting Ticket)** encrypted with a key derived from the user's password — user's actual password is NEVER sent over the network, only used locally to decrypt this response.
2. **Ticket Granting Server (TGS) step:** When user wants to access a specific service (e.g., a file server), client presents the TGT to the **TGS** and requests a **service ticket** for that specific resource.
3. **Service access step:** Client presents this service ticket directly to the target service (file server). Service decrypts it (it shares a secret key with the KDC) and trusts the identity claim inside — user is granted access, no password ever changed hands with the service either!

**Key Terms to remember:**

- **KDC (Key Distribution Center)** — the trusted third party, contains both AS and TGS
- **TGT (Ticket Granting Ticket)** — "proof you logged in", used to get further tickets, has limited lifetime (commonly 8-10 hours)
- **Service Ticket** — "proof you can access THIS specific service"
- **Realm** — Kerberos' term for a security domain (like a Windows AD domain)

**Real Example:** Logging into a corporate Windows laptop that's joined to an Active Directory domain — the moment you log in, you get a TGT automatically. After that, opening a shared network drive, printer, or internal SharePoint doesn't ask for your password again — Kerberos tickets are silently handling authentication behind the scenes.

**Security Issues / Attacks:**

- **Pass-the-Ticket** — attacker steals a valid Kerberos ticket (TGT or service ticket) from memory and reuses it to impersonate the user without needing the password at all
- **Golden Ticket Attack** — attacker who compromises the KDC's master key (`krbtgt` account hash) can forge TGTs for ANY user, including fake/nonexistent ones, with any privileges — a catastrophic, persistent domain compromise
- **Silver Ticket Attack** — similar but forges a service ticket for a SPECIFIC service (using that service account's hash), more limited but still dangerous, and harder to detect since it doesn't touch the KDC
- **Kerberoasting** — attacker requests service tickets for accounts with weak service account passwords, then cracks the ticket's encryption offline to recover the plaintext password
- **Clock skew issues** — Kerberos relies heavily on synchronized time between client/server (usually max 5 min skew allowed); if attacker can manipulate time, replay attacks become easier

**Prevention:**

- Rotate the `krbtgt` account password regularly (mitigates Golden Ticket persistence)
- Strong, random passwords for service accounts (defeats Kerberoasting)
- Monitor for abnormal ticket requests/replay patterns
- Enforce short ticket lifetimes where feasible
- Keep domain controllers strictly time-synchronized (NTP)

**💬 Interview One-Liner:** "Kerberos uses a ticket-based system through a trusted KDC so passwords never cross the network — but if an attacker compromises the KDC's master key, they can forge a Golden Ticket and impersonate anyone in the domain indefinitely."

---

## 9. LDAP

**What it is:** LDAP (Lightweight Directory Access Protocol) isn't itself an authentication _method_ like the others — it's a protocol for reading/writing data from a **directory service** (a hierarchical database of users, groups, computers). It's most often used as the **backend store** that other authentication systems (Kerberos/AD, apps doing "bind" auth) check against.

**How it works (as used for authentication — "LDAP Bind" auth, most commonly asked flow):**

1. App collects username + password from user (a normal login form)
2. App connects to the LDAP server and attempts a **"bind"** operation — essentially trying to log into the directory itself using the provided username (mapped to a Distinguished Name, DN) and password
3. If the LDAP server accepts the bind (i.e., the password is correct for that DN), authentication succeeds
4. App can then optionally **search** the directory for that user's additional attributes (email, group memberships, roles) to make authorization decisions

**Key Terms to remember:**

- **DN (Distinguished Name)** — the full, unique path/identifier of an entry, e.g. `cn=john,ou=employees,dc=company,dc=com`
- **CN, OU, DC** — Common Name, Organizational Unit, Domain Component — building blocks of a DN
- **Directory Tree** — LDAP data is structured hierarchically (like a file system tree), not in flat tables like SQL

**Real Example:** Many corporate intranet apps, VPN logins, and internal tools authenticate by "binding" against the company's Active Directory (which itself exposes an LDAP interface) — this is why IT departments say "just use your AD/LDAP credentials" for internal tools.

**Security Issues / Attacks:**

- **LDAP Injection** — covered in detail in the Application Layer Attacks note; unescaped user input manipulates the LDAP filter/query logic to bypass authentication
- **Unauthenticated / Anonymous Bind** — some LDAP servers are misconfigured to allow binding with an empty password, which some libraries incorrectly treat as "successful login" instead of rejecting it (a classic real bug: empty password + username somehow returns "success" because an anonymous bind isn't the same as failure)
- **Plaintext LDAP (port 389, no TLS)** — credentials sent in cleartext over the network, sniffable
- **No account lockout at the LDAP layer** — directory itself might not throttle failed bind attempts, enabling brute force if the app doesn't add its own layer

**Prevention:**

- Always use **LDAPS (LDAP over TLS/SSL)**, never plaintext LDAP
- Explicitly reject empty-password bind attempts in application code (don't just trust "bind succeeded")
- Escape/sanitize all user input before building LDAP filter queries
- Apply account lockout/rate limiting at the application layer, don't rely solely on directory defaults

**💬 Interview One-Liner:** "LDAP itself is just a directory protocol, not an auth method — but 'LDAP Bind' authentication is extremely common in enterprises, and the classic gotcha is that an empty password can trigger an anonymous bind that some buggy code mistakes for a successful login."

---

## Comparison Table - Which One Is Used Where

|Technology|Primary Use Case|Stateful?|Format|Typical Real-World Example|
|---|---|---|---|---|
|**OAuth2**|Delegated authorization (access to resources)|Depends on impl.|JSON / tokens|"Allow app to access your Google Calendar"|
|**OIDC**|Authentication (identity) on top of OAuth2|Stateless (ID token)|JWT|"Sign in with Google"|
|**JWT**|Token format for stateless auth/claims|Stateless|Base64 JSON (3 parts)|API auth headers (`Bearer <token>`)|
|**Session Auth**|Traditional web login|Stateful|Random session ID + server store|Classic web apps, banking portals|
|**MFA**|Extra verification layer (not a full auth system)|N/A|OTP / Push / Biometric|Bank login asking for OTP after password|
|**Passkeys**|Passwordless login|Stateless (public/private key)|FIDO2/WebAuthn keys|"Sign in with Face ID"|
|**SAML**|Enterprise SSO|Stateless (per assertion)|XML|Corporate Okta/Azure AD SSO to Salesforce|
|**Kerberos**|Network/domain authentication|Ticket-based (stateful tickets, stateless verification)|Binary tickets|Windows Active Directory domain login|
|**LDAP**|Directory service (backend for auth)|N/A (protocol)|LDAP entries/DN|Corporate intranet tools checking AD credentials|

---

## Quick Revision Table (Night Before Interview)

|#|Topic|One-Line Core Idea|Common Attack|Fix|
|---|---|---|---|---|
|1|OAuth2|Delegated authorization, not authentication|CSRF via missing `state`, open redirect|Validate `redirect_uri` + `state` + PKCE|
|2|OIDC|Adds identity (ID Token/JWT) on top of OAuth2|Forged identity via bad token validation|Verify signature, `iss`, `aud`, `nonce`|
|3|JWT|Signed, self-contained token (NOT encrypted)|`alg:none`, algorithm confusion, weak secret|Hardcode algorithm, strong secret, short expiry|
|4|Session Auth|Stateful — server stores session, client holds ID|Session fixation/hijacking|Regenerate ID on login, `HttpOnly`/`Secure`|
|5|MFA|2+ factors from different categories|SIM swap, MFA fatigue/push bombing|Prefer TOTP/hardware keys, number matching|
|6|Passkeys|Public-key crypto, private key never leaves device|Device loss, downgrade to password fallback|Multiple backup passkeys, remove weak fallback|
|7|SAML|XML-based enterprise SSO via signed Assertion|XML Signature Wrapping, XXE|Strict signature validation, disable DTD|
|8|Kerberos|Ticket-based auth via KDC, password never on wire|Golden Ticket, Kerberoasting|Rotate krbtgt, strong service account passwords|
|9|LDAP|Directory protocol, "bind" = login attempt|LDAP Injection, anonymous bind mistaken as login|Use LDAPS, escape input, reject empty password bind|

---

## Bonus Tip for Interview 😄

Ekta common trick question interviewers ask: **"What's the difference between OAuth2 and OIDC?"** — most students mix these up. Say this confidently:

> "OAuth2 answers 'what can this app access', OIDC answers 'who is this user' — OIDC is literally built as a thin identity layer ON TOP of OAuth2, and the giveaway is the ID Token, which is a JWT that OAuth2 alone never returns."

Another good one: **"Why can't you just revoke a JWT like a session?"** — this shows you understand the stateless vs stateful tradeoff deeply:

> "Because a JWT is self-contained and the server doesn't track it anywhere — the server would need to check a deny-list on every request anyway, which defeats the whole 'stateless' benefit JWT was supposed to give you."

Bhai, if you can explain the **tradeoffs** (not just definitions) between these systems, that's exactly what separates a "memorized" candidate from someone who actually _understands_ auth. All the best! 🚀