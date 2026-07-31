# 05 - Authorization

## Contents

- [[RBAC]]
- [[ABAC]]
- [[PBAC]]
- [[Least Privilege]]
- [[Zero Trust]]
- [[Permission Escalation]]
- [[Privilege Escalation]]
---

## title: "05 - Authorization" tags: [cybersecurity, appsec, authorization, interview-prep, web-security] created: 2026-07-16

# 05 - Authorization

> **Quick context before we start, bhai:** Remember from the Authentication note: **Authentication = "who are you"**, **Authorization = "what are you allowed to do"**. This note is all about the second part — the actual RULES and MODELS that decide what a logged-in user can/cannot access or do.
> 
> Same structure for every topic:
> 
> - 🔹 **What it is**
> - 🔹 **How it works**
> - 🔹 **Real Example**
> - 🔹 **Security Issues / Attacks**
> - 🔹 **💬 Interview One-Liner**

---

## Table of Contents

1. [[#1. RBAC (Role-Based Access Control)]]
2. [[#2. ABAC (Attribute-Based Access Control)]]
3. [[#3. PBAC (Policy-Based Access Control)]]
4. [[#4. Least Privilege]]
5. [[#5. Zero Trust]]
6. [[#6. Permission Escalation]]
7. [[#7. Privilege Escalation]]
8. [[#Comparison Table - RBAC vs ABAC vs PBAC]]
9. [[#Quick Revision Table (Night Before Interview)]]

---

## 1. RBAC (Role-Based Access Control)

**What it is:** The most common authorization model — permissions are assigned to **roles**, and users get access by being assigned to one or more roles. You never assign permissions directly to a user; you always go through a role.

**How it works:**

1. Admin defines roles: `Admin`, `Editor`, `Viewer`
2. Each role has a fixed set of permissions attached: `Admin` = create/read/update/delete, `Editor` = create/read/update, `Viewer` = read only
3. User is assigned one (or more) of these roles
4. When user tries an action, system checks: "does ANY of this user's roles have permission for this action?"

**Real Example:** A hospital management system — `Doctor` role can view/edit patient records, `Nurse` role can view records but not edit diagnosis, `Receptionist` role can only view appointment schedules. New employee joins as a nurse → just assign the `Nurse` role, done — no need to configure individual permissions one by one.

**Why it's popular:** Simple to understand, easy to audit ("show me everyone with Admin role"), maps naturally to organizational structure (job titles).

**Security Issues / Attacks:**

- **Role Explosion** — too many fine-grained roles created over time (`Editor-East-Region`, `Editor-West-Region-Weekend`...) becomes unmanageable, defeats the simplicity RBAC was meant to give
- **Static roles can't handle context** — RBAC alone can't express "Editor can only edit during business hours" or "Doctor can only see THEIR OWN patients" — this is RBAC's core limitation (leads into ABAC below)
- **Role assignment errors** — accidentally assigning `Admin` instead of `Viewer` due to messy UI/scripts, or old employees keeping roles after department transfer (privilege creep)
- **Broken Access Control** if the app only checks role in the UI, not server-side, on every request (classic OWASP #1 issue)

**Prevention:**

- Regular access reviews / role audits (quarterly "who has what role" check)
- Enforce role checks server-side on every request, not just UI hiding
- Keep roles broad and few; use ABAC/PBAC for fine-grained, contextual rules on top
- Automate role revocation on department transfer/offboarding (don't rely on manual process)

**💬 Interview One-Liner:** "RBAC assigns permissions to roles, not directly to users, which makes it simple to manage and audit — but it breaks down when you need context-aware rules, like 'a doctor can only see their own patients', which is exactly where ABAC comes in."

---

## 2. ABAC (Attribute-Based Access Control)

**What it is:** A more flexible, fine-grained authorization model where access decisions are made by evaluating **attributes** — of the user, the resource, the action, and the environment — against a policy, in real time.

**How it works:** Decision is based on a combination of attribute categories:

- **Subject attributes** (the user): department, clearance level, job title
- **Resource attributes** (the object being accessed): document classification, owner, project ID
- **Action attributes**: read, write, delete, approve
- **Environment attributes**: time of day, IP address/location, device type

A policy engine evaluates a rule like:

```
ALLOW if subject.department == resource.department 
       AND subject.clearance >= resource.classification 
       AND environment.time BETWEEN 9AM-6PM
```

Every access request is evaluated **dynamically** against these attributes — nothing is pre-assigned like a static role.

**Real Example:** A cloud file-sharing system where the rule is: "Finance department employees can access Finance folder documents ONLY during business hours AND only from a company-managed device." This can't be expressed cleanly with RBAC alone (you'd need dozens of time-based/device-based roles) but is a single, natural ABAC policy.

**Security Issues / Attacks:**

- **Policy complexity** — as rules grow, they become hard to test/audit/debug ("why was this specific request denied?" can require tracing through many attribute combinations)
- **Attribute integrity** — if attacker can forge/tamper with an attribute value (e.g., spoof their department in a JWT claim, spoof location via VPN), the whole access decision is compromised — ABAC is only as trustworthy as its attribute sources
- **Performance overhead** — evaluating multiple attributes dynamically on every single request is more expensive than a simple role lookup
- **Policy conflicts** — two rules might contradict each other (one says allow, another says deny) — needs clear conflict-resolution logic (e.g., "deny always wins")

**Prevention:**

- Ensure attribute sources are trustworthy/verified (signed claims, server-side attribute lookup — never trust client-supplied attributes directly)
- Centralize policy management (don't scatter ABAC logic across different services inconsistently)
- Test policies thoroughly, have clear "deny overrides allow" conflict resolution
- Cache attribute lookups carefully for performance, without going stale on critical attributes

**💬 Interview One-Liner:** "ABAC makes access decisions dynamically from attributes of the user, resource, action, and environment — it's far more flexible than RBAC for context-aware rules, but it's also only as secure as the trustworthiness of those attributes themselves."

---

## 3. PBAC (Policy-Based Access Control)

**What it is:** An authorization approach where access decisions are driven by **centrally defined, human-readable policies** (often written in a dedicated policy language) evaluated by a dedicated **policy engine** — think of it as ABAC's more formalized, enterprise-grade big brother, often built USING attributes but organized around explicit business-rule policies.

> 💡 **Interview note:** PBAC and ABAC overlap a LOT in real-world discussion, and different sources define them slightly differently. The safest way to explain the distinction: **ABAC is about the "ingredients" (attributes) used in a decision. PBAC is about the "recipe" (the centrally-managed policy/rule engine) that combines those ingredients into an enterprise-wide, auditable decision framework.** Many real ABAC systems are implemented AS a PBAC architecture.

**How it works:**

1. Organization defines policies in a structured/declarative language (e.g., **XACML**, or modern tools like **OPA - Open Policy Agent** with its **Rego** language)
2. Policies are stored centrally in a **Policy Administration Point (PAP)**
3. When an access request comes in, it goes to a **Policy Enforcement Point (PEP)** — usually sitting right at the application/API gateway
4. PEP sends the request details to a **Policy Decision Point (PDP)** — the actual policy engine that evaluates the request against all applicable policies
5. PDP may consult a **Policy Information Point (PIP)** to fetch additional attributes (user's department from HR system, resource classification from a data catalog) needed to evaluate the policy
6. PDP returns `Permit` / `Deny` (and sometimes `Not Applicable` / `Indeterminate`) back to PEP, which enforces the decision

**Real Example:** A large bank uses **Open Policy Agent (OPA)** across all its microservices — instead of writing authorization `if` statements inside every single service's code, all services call out to a central OPA instance with a Rego policy like: "Deny any transaction approval request if the approver is the same person who submitted the transaction" — one policy, enforced consistently everywhere, easily auditable by the compliance team.

**Security Issues / Attacks:**

- **PDP as single point of failure/bottleneck** — if the central policy engine goes down or is slow, it can break authorization across the entire system (availability risk)
- **Policy misconfiguration** — a wrongly written policy (typo in Rego, wrong operator) can silently over-permit or over-deny across the WHOLE organization at once, since it's centralized — blast radius is much larger than a single service's local bug
- **PEP bypass** — if a service calls the actual resource directly without going through its PEP (developer shortcut/misconfiguration), the whole policy layer is skipped entirely
- **Stale attribute data from PIP** — if HR system attribute update is delayed, wrong access decisions get made based on outdated info

**Prevention:**

- Version control + peer review + automated testing for all policy changes (treat policies like code — "Policy as Code")
- High availability / caching for the PDP so it's not a bottleneck
- Ensure EVERY code path goes through the PEP — no direct resource access shortcuts
- Monitor/alert on policy evaluation failures and PDP downtime

**💬 Interview One-Liner:** "PBAC centralizes authorization logic into explicit, auditable policies evaluated by a dedicated engine — like OPA — using a PEP/PDP/PIP architecture, which is powerful for consistency across microservices, but a single bad policy change can silently misconfigure access across the entire organization at once."

---

## 4. Least Privilege

**What it is:** A foundational security **principle** (not a specific technology) stating that every user, process, or system should be granted the **minimum level of access** necessary to perform its function — nothing more.

**How it works in practice:**

- Grant access **only** to what's needed for the current task/job function
- Grant access **only for as long as needed** (time-bound/temporary elevation instead of permanent access — this is often called **Just-In-Time (JIT) access**)
- Regularly review and **revoke unused permissions** (permissions tend to accumulate over time — "privilege creep" — as people change roles but old access never gets removed)
- Apply to EVERYTHING: human users, service accounts, API keys, applications, even network segments

**Real Example:** A backend service that only needs to READ from a database table to generate reports should be given a DB credential with **SELECT-only** permission — NOT a full admin DB user. If that service gets compromised (e.g., via SQL Injection), the attacker is stuck with read-only access instead of being able to drop tables or modify data.

**Why interviewers ask this a lot:** It's the single most-referenced principle across almost EVERY other topic in this whole doc — IDOR, Broken Access Control, Command Injection blast radius, Kerberos service accounts, cloud IAM roles — least privilege is the "default deny, grant minimally" philosophy underlying all of them.

**Security Issues / Violations (what happens when it's NOT followed):**

- **Privilege creep** — employees accumulate permissions across multiple role changes over years, nobody removes old access
- **Overly broad service accounts/API keys** — e.g., a CI/CD pipeline given full cloud admin rights when it only needs to deploy to one specific S3 bucket
- **Shared/generic accounts with high privilege** — makes it impossible to audit "who did this action" and increases blast radius if compromised
- **Larger blast radius on compromise** — if an over-privileged account/service is breached, attacker inherits ALL of that excess access

**Prevention:**

- Default to **deny-all**, explicitly grant only what's justified
- Regular **access reviews/audits** (quarterly, or automated continuous review)
- Use **Just-In-Time (JIT) elevation** for admin tasks (request temporary elevated access, auto-expires) instead of standing admin access
- Separate accounts for different privilege levels (e.g., a sysadmin's daily-use account vs their break-glass admin account)

**💬 Interview One-Liner:** "Least Privilege means granting only the minimum access needed, for only as long as it's needed — it's the underlying philosophy behind almost every access control failure in security, because most real breaches become catastrophic specifically due to excess, unused privilege."

---

## 5. Zero Trust

**What it is:** A modern security **architecture/philosophy** built on the core assumption: **"Never trust, always verify"** — no user, device, or request is trusted by default, regardless of whether it's coming from inside or outside the traditional network perimeter.

> ⚠️ **Interview trap:** Zero Trust is NOT a single product/technology you buy — it's an architectural philosophy/framework (often called **ZTA - Zero Trust Architecture**) that combines MANY of the other things in this note (least privilege, strong authentication, continuous verification) together.

**How it works — Core Principles (memorize these, commonly asked):**

1. **Verify explicitly** — always authenticate and authorize based on ALL available data points (identity, device health, location, service, workload) for EVERY request, not just once at login
2. **Use least privilege access** — just-in-time and just-enough-access, risk-based adaptive policies
3. **Assume breach** — design the system as if an attacker is ALREADY inside the network; segment access so a breach in one area can't spread everywhere (**micro-segmentation**), encrypt everything, minimize blast radius

**The old model vs Zero Trust (classic comparison to mention):**

|Traditional "Castle-and-Moat" Model|Zero Trust Model|
|---|---|
|Trust anything inside the network perimeter (firewall/VPN)|Trust NOTHING by default, verify every single request|
|One-time authentication at network entry (VPN login)|Continuous, per-request verification|
|Flat internal network — lateral movement easy once inside|Micro-segmented — compromise of one resource doesn't grant access to others|

**Real Example:** Google's internal **BeyondCorp** model (the famous origin case study of Zero Trust) — Google employees can access internal tools from ANY network (even public coffee shop WiFi) without a VPN, because every single request is independently verified based on user identity + device security posture + context, rather than trusting "you're on the corporate network, so you're fine."

**Security Issues / Common Mistakes:**

- **"Zero Trust-washing"** — companies claim Zero Trust just by adding MFA or a new VPN product, without actually implementing continuous verification or micro-segmentation (interviewer will respect you calling this out!)
- **Incomplete coverage** — implementing Zero Trust for user access but leaving service-to-service (machine) communication still implicitly trusted
- **Over-reliance on device posture alone** — if device health checks can be spoofed/bypassed, the "verify explicitly" principle breaks down
- **Poor UX from excessive re-verification** — done badly, causes friction/annoyance leading users to find workarounds

**Prevention/Implementation Best Practices:**

- Strong identity verification (MFA, ideally passkeys) as the foundation
- Micro-segmentation of network/application access (don't allow lateral movement)
- Continuous monitoring + risk-based adaptive access (step-up authentication for risky actions)
- Apply Zero Trust to service-to-service (workload) communication too, not just human users

**💬 Interview One-Liner:** "Zero Trust is 'never trust, always verify' — instead of trusting anything inside the network perimeter, every single request is independently verified based on identity, device, and context, with micro-segmentation limiting blast radius under the assumption a breach has already happened."

---

## 6. Permission Escalation

**What it is:** When a user or process gains access to permissions/capabilities **beyond what was explicitly granted to them** — but strictly WITHIN the same general level (as opposed to Privilege Escalation, which specifically implies jumping to a HIGHER trust/privilege TIER like admin/root — covered next).

> 💡 **Interview note:** In casual industry usage, "Permission Escalation" and "Privilege Escalation" are often used interchangeably — but if the interviewer wants precision, the safest distinction is: **Permission Escalation = gaining unauthorized specific permissions/access rights (broader term, can be horizontal or vertical)**, while **Privilege Escalation = specifically gaining a higher PRIVILEGE LEVEL/tier (vertical only, e.g., user → admin, or normal process → root/SYSTEM)**. We'll draw this distinction clearly in both sections.

**How it works:**

- Exploiting **Broken Access Control** or **IDOR**-style bugs (already covered in Application Layer Attacks note) to access permissions meant for a different user/resource
- **Horizontal escalation** — gaining access to another user's SAME-LEVEL resources (e.g., normal User A viewing normal User B's private data) — same privilege tier, just wrong target
- Exploiting **Mass Assignment** (covered earlier) to set a `permissions` field directly during a request
- Chaining smaller permission grants together in unintended ways ("permission creep" abused deliberately by an attacker who already has SOME legitimate low-level access)

**Real Example:** A SaaS project management tool where User A (assigned to Project X only) discovers that changing the `project_id` parameter in an API call lets them view/edit tasks in Project Y, which they were never invited to — they didn't become an "admin", they just gained permissions on a resource outside their granted scope. This is a classic IDOR-flavored Permission Escalation.

**Security Issues / Attacks:**

- Same root causes as **Broken Access Control** and **IDOR** — server-side checks missing or incomplete
- **API endpoints with inconsistent authorization** — the web UI enforces permission checks correctly, but a lesser-used API endpoint (mobile app backend, internal admin tool, legacy endpoint) forgot the same check
- **Insufficiently scoped tokens/API keys** — a token meant for "read own profile" accidentally also works for "read any profile" because the backend didn't scope the check to the token's intended resource

**Prevention:**

- Enforce object-level and function-level authorization checks on EVERY endpoint, consistently
- Scope API tokens/keys tightly to specific resources/actions, verify scope server-side on every call
- Centralize authorization logic (see PBAC above) instead of scattering ad-hoc checks across many endpoints, which is exactly how inconsistencies creep in
- Regular API security testing focused specifically on authorization (not just authentication)

**💬 Interview One-Liner:** "Permission Escalation is gaining access rights beyond what was granted, often horizontally — like one normal user reaching another user's data — and it's almost always caused by inconsistent or missing server-side authorization checks across different endpoints."

---

## 7. Privilege Escalation

**What it is:** Specifically gaining access to a **higher privilege/trust TIER** than originally granted — the classic "vertical" jump, e.g., a normal user becoming an admin, or a low-privilege process gaining root/SYSTEM-level control of the underlying machine.

**How it works — Two main types (memorize this pair, very commonly asked):**

- **Vertical Privilege Escalation** — moving to a HIGHER privilege level (normal user → admin, or web app user → root shell on the server). This is the "textbook" definition of Privilege Escalation.
- **Horizontal Privilege Escalation** — accessing another account/resource at the SAME privilege level (this overlaps heavily with what we called "Permission Escalation" above — again, terminology overlaps in the industry, so mention both terms when discussing this to show you know the nuance!)

**Common Real-World Techniques (great to mention specific ones in interview):**

- **Exploiting Broken Access Control / IDOR** to reach admin-only functionality directly (e.g., accessing `/admin/panel` URL directly without a role check)
- **OS-level exploitation** — exploiting an unpatched kernel vulnerability or misconfigured SUID binary on Linux to go from a regular shell to root
- **Exploiting misconfigured cloud IAM roles** — a compromised low-privilege service (e.g., via SSRF, covered in Application Layer Attacks) that has an attached cloud role with excessive permissions, letting attacker "escalate" simply by using that role's own excessive rights
- **Token/cookie manipulation** — modifying a JWT claim (`role: admin`) if the signature isn't properly verified (ties back to JWT Attacks from the Authentication note)
- **DLL Hijacking / Scheduled Task abuse (Windows)** — placing a malicious file where a higher-privileged process will load/execute it automatically

**Real Example:** The famous **Capital One breach (2019)** is actually a beautiful chain of BOTH concepts: attacker used **SSRF** to reach the AWS metadata endpoint from a compromised web app server (low privilege), retrieved IAM credentials attached to that server's role — and because that IAM role was **over-privileged** (violating Least Privilege!), the attacker effectively "escalated" to having broad S3 access across the company's data, without ever touching a traditional "admin panel."

**Security Issues / Attacks:** _(the techniques above ARE the attacks — summarizing root causes)_

- Missing/broken server-side authorization on privileged actions
- Unpatched systems (kernel exploits, known CVEs)
- Over-privileged service roles/accounts (violates Least Privilege directly — these two topics are deeply connected!)
- Trusting client-side data (JWT claims, cookies) for privilege decisions without proper signature verification

**Prevention:**

- Strict server-side role/privilege verification on every sensitive action — never trust client input for privilege level
- Regular patching (OS, libraries, dependencies) to close known escalation exploits
- Apply **Least Privilege** rigorously to service accounts/cloud IAM roles (directly prevents the "SSRF → escalate via over-privileged role" pattern)
- Principle of separation: admin functionality on a completely separate, more heavily guarded interface/subdomain, not just a hidden URL
- Monitor for unusual privilege usage patterns (a service account suddenly accessing resources it's never touched before)

**💬 Interview One-Liner:** "Privilege Escalation specifically means jumping to a higher trust tier — vertical, like user-to-admin or app-to-root — and the Capital One breach is the textbook real-world example, where SSRF combined with an over-privileged IAM role let the attacker escalate without ever touching an admin panel."

---

## Comparison Table - RBAC vs ABAC vs PBAC

|Model|Decision Based On|Flexibility|Complexity|Best For|
|---|---|---|---|---|
|**RBAC**|Static role assignment|Low|Low|Simple orgs, clear job-title-based access|
|**ABAC**|Dynamic attributes (user/resource/env)|High|Medium-High|Context-aware rules (time, location, device)|
|**PBAC**|Centralized, explicit policies (often using attributes)|High|High (but centrally managed/auditable)|Enterprise-wide consistency across many microservices|

---

## Quick Revision Table (Night Before Interview)

|#|Topic|One-Line Core Idea|Common Failure Mode|Fix|
|---|---|---|---|---|
|1|RBAC|Permissions attached to roles, not users|Role explosion, static rules can't handle context|Keep roles broad, layer ABAC/PBAC on top|
|2|ABAC|Real-time decision from attributes|Attribute forgery/spoofing|Never trust client-supplied attributes|
|3|PBAC|Centralized policy engine (PEP/PDP/PIP)|PEP bypass, bad policy = org-wide impact|Policy-as-code with review/testing|
|4|Least Privilege|Minimum access, minimum time|Privilege creep over time|JIT access + regular access reviews|
|5|Zero Trust|Never trust, always verify every request|"Zero Trust-washing" (just adding MFA)|True continuous verification + micro-segmentation|
|6|Permission Escalation|Gaining access beyond granted scope (often horizontal)|Inconsistent authorization across endpoints|Centralize + test authorization on every endpoint|
|7|Privilege Escalation|Jumping to a higher trust tier (vertical)|Over-privileged service roles, unpatched systems|Least privilege on service accounts + patching|

---

## Bonus Tip for Interview 😄

Interviewer's favorite trick question here: **"What's the difference between Permission Escalation and Privilege Escalation?"** Most people don't even know these are considered separately — answer with confidence:

> "Privilege Escalation is specifically vertical — jumping to a higher trust tier, like user to admin. Permission Escalation is the broader term that also covers horizontal cases, like one normal user reaching another user's data at the same tier. In practice the industry uses them loosely, but I always clarify which direction — vertical or horizontal — when discussing a specific finding."

Also, always connect **Least Privilege** and **Zero Trust** back to real breaches when asked "why does this matter" — the Capital One example (SSRF + over-privileged IAM role) is gold because it ties together SSRF, Privilege Escalation, AND violation of Least Privilege all in one famous, well-documented case. Interviewers love candidates who can connect concepts across topics instead of reciting isolated definitions. Best of luck, bhai! 🚀