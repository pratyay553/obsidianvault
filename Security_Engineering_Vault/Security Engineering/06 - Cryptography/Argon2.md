# Argon2

## Definition

**Argon2** is a modern **password hashing algorithm** designed to securely store passwords by making them computationally expensive to crack. It won the **Password Hashing Competition (PHC)** in **2015** and is widely recommended by security experts, including **OWASP**.

> **Key Idea:** **Passwords should be hashed, not encrypted.** Hashing is a **one-way process**—you can verify a password but cannot recover the original password from its hash.

### Simple Analogy

Think of a **paper shredder**:

- You put a document (password) into the shredder.
    
- It produces tiny pieces (hash).
    
- You **cannot reconstruct** the original document.
    
- But if you shred the same document again using the same settings, you'll get a matching result for verification.
    

---

# Why it Matters

Passwords are one of the most common attack targets.

If passwords are stored as plain text:

```text
john123
admin@123
password
```

A database breach exposes every user's password immediately.

Instead, store:

```text
$argon2id$v=19$m=65536,t=3,p=4$...
```

Even if attackers steal the database, recovering passwords becomes extremely difficult.

### Benefits

- Extremely resistant to brute-force attacks
    
- Designed to defeat GPU attacks
    
- Memory-hard (requires significant RAM)
    
- Automatically uses random salts
    
- Recommended by OWASP
    
- More secure than older algorithms like MD5 and SHA-1 for password storage
    

---

# How it Works

## Step 1: User Enters Password

```text
MyPassword@123
```

---

## Step 2: Generate a Random Salt

Example:

```text
Salt = x9P4Lm8Q
```

Each user gets a unique random salt.

---

## Step 3: Argon2 Hashing

Argon2 combines:

- Password
    
- Salt
    
- Memory cost
    
- Iterations
    
- Parallelism
    

and performs intensive computations.

Example output:

```text
$argon2id$v=19$m=65536,t=3,p=4$
c2FsdA$
Fsd89KJD83jd93kd...
```

---

## Step 4: Store the Hash

Only store:

- Hash
    
- Salt
    
- Parameters
    

Never store the original password.

Database example:

|Username|Password Hash|
|---|---|
|Alice|`$argon2id$v=19$m=65536,t=3,p=4$...`|

---

## Step 5: Login Verification

User enters:

```text
MyPassword@123
```

The application:

1. Reads the stored hash.
    
2. Extracts the salt and parameters.
    
3. Recomputes the hash.
    
4. Compares the new hash with the stored hash.
    

If they match:

```text
Login Successful
```

Otherwise:

```text
Invalid Password
```

---

## Argon2 Flow Diagram

```text
User Password
      │
      ▼
Generate Random Salt
      │
      ▼
Argon2 Hashing
(memory + iterations)
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

# How Argon2 Protects Against Attacks

### Brute Force Attack

Without hashing:

```text
password123
```

Attackers instantly know the password.

With Argon2:

```text
$argon2id$...
```

Each guess is slow and expensive.

---

### Rainbow Table Attack

Argon2 automatically uses **random salts**, making precomputed rainbow tables ineffective.

---

### GPU Attack

Argon2 requires large amounts of memory.

GPUs are optimized for computation but are less efficient for memory-intensive workloads, making large-scale password cracking significantly slower.

---

# Variants of Argon2

|Variant|Purpose|
|---|---|
|**Argon2d**|Maximum resistance to GPU cracking; suited for cryptocurrencies and specialized use cases.|
|**Argon2i**|Protects against side-channel attacks; older recommendation for password hashing.|
|**Argon2id**|Combines the advantages of Argon2d and Argon2i; **recommended for password hashing**.|

> **Use Argon2id** for most applications.

---

# Common Pitfalls

## 1. Using MD5 or SHA-1

❌ Bad

```text
MD5(password)
```

These are fast hash functions and are unsuitable for password storage.

✔ Use Argon2id.

---

## 2. Using SHA-256 Directly

```java
MessageDigest.getInstance("SHA-256");
```

SHA-256 is excellent for integrity checks but **too fast** for password hashing.

---

## 3. Low Memory Settings

Weak:

```text
Memory = 1 MB
```

Better:

```text
Memory = 64 MB
```

Higher memory makes attacks significantly more expensive.

---

## 4. Low Iteration Count

Weak:

```text
Iterations = 1
```

Better:

```text
Iterations = 3 or more
```

Increasing iterations raises the computational cost per password guess.

---

## 5. Hardcoding a Salt

❌ Bad

```text
Salt = abc123
```

Each password should use a **new random salt**.

---

## 6. Encrypting Passwords Instead of Hashing

Never encrypt passwords for storage.

Always hash them using Argon2id, BCrypt, SCrypt, or PBKDF2.

---

# Spring Boot / Java Notes

## Maven Dependency (Bouncy Castle Example)

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk18on</artifactId>
    <version>1.78.1</version>
</dependency>
```

---

## Spring Security (Recommended)

```java
PasswordEncoder encoder = new Argon2PasswordEncoder();

String hash = encoder.encode(password);
```

---

## Verify Password

```java
boolean match = encoder.matches(rawPassword, storedHash);
```

---

## Spring Bean

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new Argon2PasswordEncoder();
}
```

---

## Login Example

```java
if(passwordEncoder.matches(password, user.getPassword())) {
    // Login successful
}
```

---

## Best Practices

- ✅ Use **Argon2id**
    
- ✅ Generate a random salt (handled automatically by Spring Security)
    
- ✅ Tune memory and iteration settings based on your server's capabilities
    
- ✅ Never store plain-text passwords
    
- ✅ Never implement your own hashing algorithm
    
- ✅ Periodically review and increase hashing parameters as hardware improves
    

---

# Interview Questions

### 1. What is Argon2?

Argon2 is a modern **password hashing algorithm** designed to securely store passwords using memory-hard computations.

---

### 2. Is Argon2 encryption?

**No.**

Argon2 is **hashing**, not encryption.

Hashing is one-way; encryption is reversible with a key.

---

### 3. Why is Argon2 considered secure?

Because it is:

- Memory-hard
    
- Resistant to GPU attacks
    
- Resistant to brute-force attacks
    
- Resistant to rainbow table attacks (through salting)
    

---

### 4. Which Argon2 variant should you use?

**Argon2id**

It combines resistance to both GPU attacks and side-channel attacks.

---

### 5. What is a salt?

A random value added to a password before hashing to ensure identical passwords produce different hashes.

---

### 6. Can you decrypt an Argon2 hash?

No.

Hashes are one-way and cannot be decrypted.

---

### 7. Why not use SHA-256 for passwords?

SHA-256 is designed to be fast. Fast hashes allow attackers to test billions of password guesses quickly.

---

### 8. Difference between BCrypt and Argon2?

|BCrypt|Argon2|
|---|---|
|CPU-hard|CPU + Memory-hard|
|Older algorithm|Modern PHC winner|
|Widely supported|Increasingly recommended|
|Good security|Stronger defense against GPU attacks|

---

### 9. Does Argon2 automatically generate salts?

Most implementations, including Spring Security's `Argon2PasswordEncoder`, generate and store salts automatically.

---

### 10. Can two users with the same password have different hashes?

Yes.

Different random salts produce different hashes even for identical passwords.

---

# References

- **Password Hashing Competition (PHC)** – Argon2 Winner
    
- **OWASP Password Storage Cheat Sheet**
    
- **Spring Security Documentation – `Argon2PasswordEncoder`**
    
- **RFC 9106 – Argon2 Memory-Hard Password Hash Function**
    

---

# Quick Revision (1-Minute Recap)

|Topic|Remember|
|---|---|
|Type|Password Hashing Algorithm|
|Winner|Password Hashing Competition (2015)|
|Recommended Variant|Argon2id|
|Reversible?|❌ No (one-way hashing)|
|Uses Salt|✅ Yes|
|Memory-Hard|✅ Yes|
|Protects Against|Brute-force, GPU attacks, Rainbow tables|
|Better Than|MD5, SHA-1, SHA-256 (for password storage)|
|Spring Security Class|`Argon2PasswordEncoder`|
|Password Storage|✅ Excellent choice|