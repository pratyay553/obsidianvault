---
tags: [java, spring-boot, servlet, http, spring-security, backend]
aliases: [ServletRequest, ServletResponse, Request-Response Cycle]
---

# HttpServletRequest vs HttpServletResponse

> [!abstract] Core Idea
> In every Servlet container (Tomcat, Jetty, etc.), each HTTP call is represented by **two objects**:
> - `HttpServletRequest` → everything the **client sent** to the server
> - `HttpServletResponse` → everything the **server sends back** to the client
>
> These two objects travel together through the entire request pipeline — see [[Servlet Filter Chain]] and [[Spring Security Filter Chain]].

## The Flow

```text
Client
   |
   | HTTP Request
   |-------------------->
   |                     |
   |              HttpServletRequest
   |                     |
   |              Spring Controller
   |                     |
   |              HttpServletResponse
   |                     |
   |<--------------------|
   |    HTTP Response
```

Related: [[DispatcherServlet]] · [[Spring MVC Request Lifecycle]]

---

## 1. `HttpServletRequest` — What the Client Sent

### 🌐 Client / Network Info
```java
request.getRemoteAddr();   // client IP
request.getRemotePort();   // client TCP port
request.getLocalAddr();    // server IP
request.getLocalPort();    // server port
request.getProtocol();     // e.g. HTTP/1.1
request.isSecure();        // true if HTTPS
```

> [!note] Common Gotcha
> `HttpServletRequest` **cannot** give you a MAC address — only IP + port. MAC addresses aren't part of the HTTP/TCP layer info exposed to the server. See [[Network Layers - OSI Model]] for why.

### 🔗 URL Info
For `GET https://example.com/api/orders?page=1&size=10`:

| Method | Returns |
|---|---|
| `getRequestURI()` | `/api/orders` |
| `getQueryString()` | `page=1&size=10` |
| `getMethod()` | `GET` |
| `getContextPath()` | app's context root |
| `getServletPath()` | servlet mapping path |
| `getPathInfo()` | extra path after servlet match |

### 📬 HTTP Method
`GET · POST · PUT · PATCH · DELETE · OPTIONS · HEAD`
→ See [[REST API Design]] for how these map to CRUD operations.

### 🏷️ Headers
```java
request.getHeader("User-Agent");
request.getHeader("Authorization");   // often a JWT/Bearer token — see [[JWT Authentication]]
request.getHeader("Content-Type");
```

Enumerate all:
```java
Enumeration<String> headers = request.getHeaderNames();
while (headers.hasMoreElements()) {
    String name = headers.nextElement();
    System.out.println(name + " = " + request.getHeader(name));
}
```

### ❓ Query / Form Parameters
```java
request.getParameter("id");
request.getParameterMap();
request.getParameterValues("id");
```

### 📦 Request Body
```java
request.getReader();       // text
request.getInputStream();  // binary
```
In **Spring MVC**, prefer declarative binding instead of reading manually:
```java
@PostMapping("/api/orders")
public ResponseEntity<?> create(@RequestBody OrderRequest order) { ... }
```
→ Related: [[Jackson JSON Serialization]], [[Bean Validation - @Valid]]

### 🍪 Cookies
```java
Cookie[] cookies = request.getCookies();
for (Cookie c : cookies) {
    System.out.println(c.getName() + " = " + c.getValue());
}
```
→ See [[Cookies vs Sessions vs Tokens]]

### 🗂️ Session
```java
HttpSession session = request.getSession(false); // don't create if absent
if (session != null) {
    Object user = session.getAttribute("user");
}
```
→ See [[Session Management in Spring]]

### 🔐 Authentication / Principal
```java
request.getUserPrincipal();
request.isUserInRole("ADMIN");
```
Via **Spring Security**:
```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
Collection<? extends GrantedAuthority> roles = auth.getAuthorities();
```
→ Deep dive: [[SecurityContextHolder]] · [[Authentication vs Authorization]]

---

## 2. `HttpServletResponse` — What the Server Sends Back

```text
HttpServletRequest
        ↓
    Your code
        ↓
HttpServletResponse
        ↓
      Client
```

### 🚦 HTTP Status
```java
response.setStatus(HttpServletResponse.SC_OK);                   // 200
response.setStatus(HttpServletResponse.SC_CREATED);               // 201
response.setStatus(HttpServletResponse.SC_BAD_REQUEST);            // 400
response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);          // 401
response.setStatus(HttpServletResponse.SC_FORBIDDEN);             // 403
response.setStatus(HttpServletResponse.SC_NOT_FOUND);              // 404
response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR); // 500
```
→ See [[HTTP Status Codes Cheatsheet]]

### 🏷️ Response Headers
```java
response.setHeader("X-Custom-Header", "ABC");
response.addHeader("X-Custom-Header", "XYZ"); // adds another value
response.setHeader("Cache-Control", "no-cache");
```

### 📄 Content Type & Encoding
```java
response.setContentType("application/json");
response.setCharacterEncoding("UTF-8");
```

### 📦 Response Body
```java
PrintWriter writer = response.getWriter();
writer.write("Hello Client");
```
or binary:
```java
ServletOutputStream output = response.getOutputStream();
output.write(data);
```
In Spring Boot, this is normally abstracted away:
```java
@GetMapping("/api/orders")
public ResponseEntity<List<Order>> getOrders() {
    return ResponseEntity.ok().body(orders);
}
```
→ See [[ResponseEntity vs @ResponseBody]]

### 🍪 Setting Cookies
```java
Cookie cookie = new Cookie("sessionId", "ABC123");
response.addCookie(cookie);
```
Browser receives: `Set-Cookie: sessionId=ABC123`

### ↪️ Redirects
```java
response.sendRedirect("/login");
```
Browser then issues a **new request** to `/login`.
→ See [[HTTP Redirects - 301 vs 302]]

---

## 3. Side-by-Side Comparison

| Information | `HttpServletRequest` | `HttpServletResponse` |
|---|:---:|:---:|
| Direction | Client → Server | Server → Client |
| HTTP method | ✅ | ❌ |
| Client IP / port | ✅ | ❌ |
| Server IP / port | ✅ | ❌ |
| URL | ✅ | ❌ |
| Query parameters | ✅ | ❌ |
| Headers | ✅ (read) | ✅ (write) |
| Cookies | ✅ (read) | ✅ (write) |
| Body | ✅ (read) | ✅ (write) |
| HTTP status | ❌ | ✅ |
| Session | ✅ | ❌ |
| Authentication / principal | ✅ | ❌ |
| Content type | Request's type | Response's type |
| Redirect | ❌ | ✅ |

---

## 4. Where This Fits in the Spring Security Flow

```text
Client
   |
   v
Servlet Container
   |
   v
DelegatingFilterProxy          → [[DelegatingFilterProxy]]
   |
   v
FilterChainProxy               → [[FilterChainProxy]]
   |
   v
Security Filters
   |
   +---- Authentication         → [[Authentication vs Authorization]]
   |
   +---- Authorization
   |
   v
DispatcherServlet              → [[DispatcherServlet]]
   |
   v
Controller
   |
   | HTTP Response
   v
HttpServletResponse
   |
   v
Client
```

> [!important] Key Insight
> The **same** request/response objects flow through the entire filter chain — filters can inspect and mutate them *before* the controller ever runs.

A security filter might inspect:
```java
request.getRemoteAddr();
request.getHeader("Authorization");
request.getRequestURI();
```
and mutate the response:
```java
response.setStatus(401);
response.setHeader("WWW-Authenticate", "Bearer");
```

---

## 5. Practical Next Step: Request Auditing

For security/audit logging, capture in a [[Custom Servlet Filter]]:

- IP address
- URL + HTTP method
- User-Agent
- Authenticated username
- Response status
- Timestamp
- Request ID (correlation ID — see [[Distributed Tracing - Correlation IDs]])

---

## Related Notes
- [[Spring Security Filter Chain]]
- [[DispatcherServlet]]
- [[Spring MVC Request Lifecycle]]
- [[REST API Design]]
- [[JWT Authentication]]
- [[Cookies vs Sessions vs Tokens]]
- [[HTTP Status Codes Cheatsheet]]
- [[Custom Servlet Filter]]

## Open Questions / To Explore
- [ ] How does `getSession(false)` interact with stateless JWT auth?
- [ ] Difference between `sendRedirect()` and `RequestDispatcher.forward()`
- [ ] How CORS headers get set via `HttpServletResponse`
