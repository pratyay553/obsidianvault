# Least Privilege

## Definition

**Least Privilege** is a security principle where a user, process, service, or application is granted **only the minimum permissions required to perform its intended task**.

> Give every component the smallest amount of access it needs, for the shortest amount of time possible.

For example, a Spring Boot application that only needs to **read** data from a database should not have `INSERT`, `UPDATE`, or `DELETE` permissions.

### Key Characteristics

- **Minimum access** — grant only required permissions.
    
- **Minimum scope** — restrict access to specific resources.
    
- **Minimum duration** — temporary access should expire when no longer needed.
    
- **Default deny** — access should be denied unless explicitly permitted.
    
- **Separation of duties** — sensitive operations may require different roles or permissions.
    

---

## Why It Matters

Least privilege reduces the potential damage caused by:

- Compromised user accounts
    
- Stolen credentials
    
- Vulnerable applications
    
- Insider threats
    
- Malware
    
- Misconfigured services
    
- Accidental operations
    

### Example

Suppose a web application is compromised.

**Without least privilege:**

```text
Application
    ↓
Database admin account
    ↓
Can read/write/delete everything
```

An attacker who compromises the application may gain complete database control.

**With least privilege:**

```text
Application
    ↓
Restricted database account
    ↓
Can only access required tables/operations
```

The compromise is still serious, but the attacker's capabilities are significantly limited.

### Security Benefit

Least privilege primarily helps reduce **blast radius**.

```text
Compromise
    ↓
Limited permissions
    ↓
Limited accessible resources
    ↓
Reduced impact
```

---

## How It Works

Least privilege can be applied at multiple levels.

### 1. Users

Users should receive only the permissions necessary for their job.

```text
Developer
 ├── Read source code
 ├── Deploy to development
 └── No production database admin access
```

### 2. Application Services

Each service should have its own restricted identity.

```text
Order Service
    ↓
Order DB User
    ↓
SELECT / INSERT / UPDATE
    ↓
orders table
```

It should not use a shared database administrator account.

### 3. APIs

API endpoints should enforce authorization based on roles or permissions.

```text
GET /orders
    → ORDER_READ

POST /orders
    → ORDER_CREATE

DELETE /orders/{id}
    → ORDER_DELETE
```

Having a valid login should not automatically grant access to every endpoint.

### 4. Operating System

Applications should run using dedicated OS users rather than `root` or administrator accounts.

```bash
# Bad
java -jar application.jar
# running as root

# Better
java -jar application.jar
# running as a dedicated application user
```

### 5. Cloud Resources

Cloud identities should have only the permissions required by the workload.

```text
Application
    ↓
IAM Role
    ↓
Read access to required storage
```

Instead of:

```text
Application
    ↓
AdministratorAccess
    ↓
Entire cloud account
```

### 6. Temporary Access

Privileged access should ideally be granted only when required and removed afterward.

```text
Request access
      ↓
Approval
      ↓
Temporary privilege
      ↓
Perform operation
      ↓
Privilege expires
```

---

## Common Pitfalls

### 1. Using Admin Accounts for Applications

```text
Spring Boot
    ↓
DB Admin
```

This is dangerous because a vulnerability in the application can become a database-wide compromise.

**Better:**

```text
Spring Boot
    ↓
Application-specific DB user
```

---

### 2. Giving `*` Permissions

Avoid unrestricted permissions such as:

```text
READ *
WRITE *
DELETE *
ADMIN *
```

Prefer narrowly scoped permissions:

```text
orders:read
orders:create
orders:update
```

---

### 3. Confusing Authentication with Authorization

**Authentication:**

> Who are you?

**Authorization:**

> What are you allowed to do?

```text
Authentication
    ↓
User is valid
    ↓
Authorization
    ↓
Can this user perform this operation?
```

A successfully authenticated user should not automatically have unrestricted access.

---

### 4. Relying Only on Frontend Restrictions

Hiding a button does **not** provide security.

```javascript
// Not security
if (isAdmin) {
    showDeleteButton();
}
```

The backend must enforce the permission:

```text
DELETE /users/123
        ↓
Backend authorization check
        ↓
USER_DELETE permission?
        ↓
Allow / Deny
```

---

### 5. Shared Credentials

Avoid using one powerful account across multiple services.

```text
Service A ─┐
Service B ─┼──→ admin-account
Service C ─┘
```

If the credential is compromised, every service is affected.

Prefer:

```text
Service A → service-a-account
Service B → service-b-account
Service C → service-c-account
```

---

### 6. Excessive Permissions That Accumulate

A user may start with a small set of permissions and gradually accumulate more.

```text
Role 1
  ↓
Role 2
  ↓
Temporary access
  ↓
Old access never removed
  ↓
Privilege creep
```

Regular access reviews help prevent this.

---

### 7. Secrets in Source Code

Avoid:

```java
String password = "admin123";
```

Use a secure secret-management mechanism instead.

```text
Application
    ↓
Secret Manager / Environment / Vault
    ↓
Credentials
```

The exact mechanism depends on the deployment environment.

---

## Spring Boot / Java Notes

Spring Boot commonly implements least privilege through **Spring Security**, authorization rules, service identities, database permissions, and secure secret management.

### Role-Based Authorization

For example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) {
    // ...
}
```

The user must have the required role before the method executes.

---

### Permission-Based Authorization

For finer-grained control:

```java
@PreAuthorize("hasAuthority('ORDER_DELETE')")
public void deleteOrder(Long id) {
    // ...
}
```

This is often more precise than creating a large number of broad roles.

```text
ROLE_ADMIN
    ↓
Many permissions

ORDER_DELETE
    ↓
One specific capability
```

---

### Enable Method Security

In modern Spring Security applications:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
}
```

Then authorization can be applied at the method level:

```java
@PreAuthorize("hasAuthority('USER_READ')")
public User getUser(Long id) {
    // ...
}
```

---

### HTTP-Level Authorization

Authorization can also be configured at the HTTP layer:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .requestMatchers("/orders/**").hasAuthority("ORDER_READ")
            .anyRequest().authenticated()
        );

    return http.build();
}
```

The important principle is:

```text
Public endpoint
    → explicitly public

Protected endpoint
    → explicitly authorized

Everything else
    → deny by default where appropriate
```

---

### Database Least Privilege

Do not connect a production application using a database administrator account.

Instead:

```text
Spring Boot
    ↓
app_user
    ↓
Only required database privileges
```

For example, conceptually:

```sql
GRANT SELECT, INSERT, UPDATE
ON orders
TO app_user;
```

If the application never deletes orders, it should not need `DELETE` permission.

---

### Separate Read and Write Access

For larger systems, read and write responsibilities can be separated.

```text
Read Service
    ↓
Read-only DB credentials

Write Service
    ↓
Read/write DB credentials
```

This reduces the impact of a compromised read-only component.

---

### Avoid Hard-Coded Secrets

Bad:

```properties
spring.datasource.username=admin
spring.datasource.password=supersecret
```

Prefer externalized secrets:

```text
Spring Boot
    ↓
Environment / Secret Manager
    ↓
Database credentials
```

Never commit production credentials to Git.

---

### Principle of Least Privilege + JWT

With JWT-based security, avoid putting overly broad permissions into tokens.

Instead of:

```json
{
  "role": "SUPER_ADMIN"
}
```

when the application only needs a specific capability, consider fine-grained authorities:

```json
{
  "authorities": [
    "ORDER_READ",
    "ORDER_CREATE"
  ]
}
```

The authorization model should match the application's actual security requirements.

---

## Interview Questions

### 1. What is the Principle of Least Privilege?

It means giving a user, process, service, or application **only the minimum permissions required** to perform its task.

---

### 2. Why is least privilege important?

It reduces the **blast radius** of security incidents.

If an attacker compromises a low-privilege service, the attacker should not automatically gain access to the entire system.

---

### 3. What is the difference between authentication and authorization?

**Authentication** determines who the user is.

**Authorization** determines what the authenticated user is allowed to do.

```text
Authentication → Who are you?
Authorization  → What can you do?
```

---

### 4. How would you implement least privilege in Spring Boot?

Possible approaches include:

- Spring Security
    
- Role-based access control
    
- Permission/authority-based authorization
    
- `@PreAuthorize`
    
- HTTP endpoint authorization
    
- Dedicated service accounts
    
- Restricted database users
    
- Externalized secrets
    
- Read-only credentials where appropriate
    
- Deny-by-default authorization
    

---

### 5. Why shouldn't a Spring Boot application use a DB admin account?

Because if the application is compromised, the attacker could inherit the database administrator's privileges.

A dedicated application account limits what the compromised application can access.

---

### 6. RBAC vs Least Privilege — are they the same?

No.

**RBAC (Role-Based Access Control)** is an authorization model.

**Least privilege** is a security principle.

RBAC can be used to implement least privilege, but simply having roles does not guarantee least privilege.

For example:

```text
ROLE_ADMIN
    ↓
Full system access
```

If most users don't need that access, assigning `ROLE_ADMIN` broadly violates least privilege.

---

### 7. What is privilege creep?

Privilege creep occurs when users or services accumulate permissions over time that they no longer require.

Example:

```text
Developer
  ↓
Gets production access temporarily
  ↓
Changes teams
  ↓
Still has production access
```

Regular access reviews and automatic expiration can reduce this problem.

---

### 8. What is deny by default?

Deny by default means access is rejected unless an explicit rule grants the required permission.

Conceptually:

```text
Request
   ↓
Permission explicitly granted?
   ├── Yes → Allow
   └── No  → Deny
```

This is safer than allowing access and trying to block known bad cases.

---

### 9. Can frontend authorization implement least privilege?

No.

Frontend restrictions improve user experience but are not a security boundary.

Authorization must ultimately be enforced on the server.

---

### 10. Give a real-world example of least privilege.

A reporting service only needs to read customer data.

Instead of giving it:

```text
Database Administrator
```

give it:

```text
REPORTING_READ
```

with access only to the required data.

If the reporting service is compromised, the attacker has fewer capabilities.

---

## References

- **OWASP — Principle of Least Privilege**
    
    - A foundational application-security principle for limiting permissions.
        
- **Spring Security Documentation**
    
    - Authorization, method security, roles, and authorities.
        
- **NIST — Least Privilege**
    
    - Security guidance and access-control concepts.
        

### Quick Revision

```text
Least Privilege
      │
      ├── Minimum permissions
      ├── Minimum scope
      ├── Minimum duration
      ├── Deny by default
      ├── Separate service accounts
      ├── Restricted DB users
      ├── Fine-grained authorization
      └── Reduce blast radius
```

### One-Line Interview Answer

> **Least privilege means giving every user, service, process, or application only the minimum access it needs to perform its job, thereby reducing the potential impact of a security compromise.**