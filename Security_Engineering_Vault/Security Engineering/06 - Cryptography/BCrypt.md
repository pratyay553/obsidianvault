# BCrypt

## Definition

**BCrypt** is a **password hashing algorithm** specifically designed to securely store passwords. It automatically generates a **unique random salt** for every password and is intentionally **slow**, making brute-force attacks much harder.

It was introduced in **1999** by **Niels Provos** and **David Mazières** and is one of the most widely used password hashing algorithms today.

> **Key Idea:** Passwords should **never be stored as plain text or encrypted**. They should be **hashed using BCrypt (or Argon2)**.

### Simple Analogy

Imagine a **safe with a time-delay lock**:

- Every password is placed into its own safe.
    
- Each safe has a **unique lock (salt)**.
    
- Opening the safe takes time (work factor).
    
- Trying millions of passwords becomes extremely slow.
    

---

# Why it Matters

If passwords are stored as plain text:

```text
john123
password
admin@123
```

Anyone who steals the database immediately knows every password.

With BCrypt:

```text
$2a$10$Xn9F4vK3mYJj7eJ2n...
```

The original password **cannot be recovered** from the hash.

### Benefits

- Automatically generates a unique salt
    
- Protects against rainbow table attacks
    
- Configurable work factor (cost)
    
- Slows down brute-force attacks
    
- Widely supported across programming languages
    
- Recommended by Spring Security
    

---

# How it Works

## Step 1: User Enters Password

```text
MyPassword@123
```

---

## Step 2: Generate a Random Salt

BCrypt automatically creates a random salt.

Example:

```text
Salt = a9XfL2mQ
```

No manual salt generation is required.

---

## Step 3: Hash the Password

BCrypt combines:

- Password
    
- Salt
    
- Cost factor
    

Example output:

```text
$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
```

The hash contains:

- Algorithm version
    
- Cost factor
    
- Salt
    
- Password hash
    

---

## Step 4: Store the Hash

Store only the BCrypt hash.

Example database:

|Username|Password Hash|
|---|---|
|Alice|`$2a$10$N9qo8uLO...`|

Never store the original password.

---

## Step 5: Login Verification

User enters:

```text
MyPassword@123
```

Application:

1. Reads the stored BCrypt hash.
    
2. Extracts the salt and cost.
    
3. Hashes the entered password again.
    
4. Compares both hashes.
    

If they match:

```text
Login Successful
```

Otherwise:

```text
Invalid Password
```

---

## BCrypt Flow Diagram

```text
User Password
      │
      ▼
Generate Random Salt
      │
      ▼
BCrypt Hashing
(Cost Factor)
      │
      ▼
Password Hash
      │
      ▼
Store in Database
      │
      ▼
Login
      │
      ▼
Hash Again
      │
      ▼
Compare Hashes
      │
      ▼
Authenticated
```

---

# Cost Factor (Work Factor)

The **cost factor** controls how long BCrypt takes to compute a hash.

Example:

|Cost|Relative Time|
|---|---|
|8|Fast|
|10|Moderate (Common Default)|
|12|Stronger Security|
|14|Very Secure but Slower|

Increasing the cost factor makes password cracking more difficult.

> A higher cost improves security but also increases login time.

---

# Common Pitfalls

## 1. Storing Plain Text Passwords

❌ Bad

```text
password123
```

✔ Always store BCrypt hashes.

---

## 2. Using MD5 or SHA-1

❌

```java
MessageDigest.getInstance("MD5");
```

MD5 and SHA-1 are too fast and vulnerable to modern attacks.

Use BCrypt or Argon2 instead.

---

## 3. Manually Creating Salts

```java
String salt = "abc123";
```

Don't do this.

BCrypt automatically generates a secure random salt.

---

## 4. Choosing a Very Low Cost

Weak:

```text
Cost = 4
```

Better:

```text
Cost = 10–12
```

Select a cost that balances security and acceptable login performance.

---

## 5. Encrypting Passwords Instead of Hashing

Passwords should be **hashed**, not encrypted.

Encryption is reversible.

Hashing is one-way.

---

## 6. Writing Your Own Password Hashing Logic

Never implement your own password hashing algorithm.

Use trusted libraries such as Spring Security.

---

# Spring Boot / Java Notes

## Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

## Create BCrypt Encoder

```java
PasswordEncoder encoder = new BCryptPasswordEncoder();
```

---

## Hash Password

```java
String hash = encoder.encode(password);
```

---

## Verify Password

```java
boolean matches = encoder.matches(rawPassword, storedHash);
```

---

## Spring Bean

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

## Authentication Example

```java
if (passwordEncoder.matches(password, user.getPassword())) {
    // Login successful
}
```

---

## Specify Cost Factor

```java
PasswordEncoder encoder = new BCryptPasswordEncoder(12);
```

Here, `12` is the BCrypt strength (cost factor).

---

## Best Practices

- ✅ Use `BCryptPasswordEncoder`
    
- ✅ Cost factor between **10–12** for most applications
    
- ✅ Never store plain-text passwords
    
- ✅ Let BCrypt generate salts automatically
    
- ✅ Use `matches()` instead of comparing hashes manually
    
- ✅ Consider migrating to Argon2id for new systems if appropriate
    

---

# BCrypt vs Argon2

|Feature|BCrypt|Argon2id|
|---|---|---|
|Type|Password Hashing|Password Hashing|
|Salt Generation|Automatic|Automatic|
|Memory-Hard|❌ No|✅ Yes|
|GPU Resistance|Good|Excellent|
|OWASP Recommendation|✅ Yes|✅ Preferred for new applications|
|Spring Support|Excellent|Excellent|
|Industry Adoption|Very High|Growing|

> **Recommendation:** BCrypt remains an excellent and widely deployed choice. For new applications, many security guidelines prefer **Argon2id** because of its stronger resistance to GPU-based attacks.

---

# Interview Questions

### 1. What is BCrypt?

BCrypt is a password hashing algorithm that securely stores passwords using a random salt and a configurable cost factor.

---

### 2. Is BCrypt encryption?

No.

BCrypt is a **one-way hashing algorithm**, not an encryption algorithm.

---

### 3. Why is BCrypt better than MD5?

BCrypt is intentionally slow and uses a unique salt for every password.

MD5 is extremely fast and vulnerable to brute-force and rainbow table attacks.

---

### 4. What is the cost factor?

The cost factor controls the computational effort required to generate a hash.

Higher cost = stronger security but slower hashing.

---

### 5. Does BCrypt automatically generate salts?

Yes.

Each password receives a unique random salt automatically.

---

### 6. Can BCrypt hashes be decrypted?

No.

Hashes are one-way and cannot be reversed.

---

### 7. Why can two identical passwords have different BCrypt hashes?

Because BCrypt generates a different random salt for each password.

---

### 8. What is the difference between BCrypt and SHA-256?

|BCrypt|SHA-256|
|---|---|
|Password hashing|General-purpose hashing|
|Slow by design|Fast|
|Automatic salt|No automatic salt|
|Suitable for passwords|Not recommended for password storage by itself|

---

### 9. How do you verify a password with BCrypt?

Use:

```java
passwordEncoder.matches(rawPassword, storedHash);
```

Spring Security extracts the salt and cost from the stored hash and performs the comparison.

---

### 10. Should you choose BCrypt or Argon2?

- **BCrypt**: Excellent, mature, and widely supported.
    
- **Argon2id**: Preferred for many new applications because it is memory-hard and offers stronger resistance to modern hardware attacks.
    

---

# References

- **Spring Security Documentation – `BCryptPasswordEncoder`**
    
- **OWASP Password Storage Cheat Sheet**
    
- **Niels Provos & David Mazières – BCrypt Paper**
    
- **OpenBSD BCrypt Documentation**
    

---

# Quick Revision (1-Minute Recap)

| Topic                 | Remember                                       |
| --------------------- | ---------------------------------------------- |
| Type                  | Password Hashing Algorithm                     |
| Introduced            | 1999                                           |
| Uses Salt             | ✅ Automatically                                |
| Reversible?           | ❌ No                                           |
| Cost Factor           | Configurable (commonly 10–12)                  |
| Protects Against      | Brute-force and Rainbow Table attacks          |
| Memory-Hard           | ❌ No                                           |
| Spring Security Class | `BCryptPasswordEncoder`                        |
| Password Verification | `matches(rawPassword, storedHash)`             |
| Best Use              | Secure password storage in modern applications |