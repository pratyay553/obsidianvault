## Definition

**AES (Advanced Encryption Standard)** is a **symmetric-key encryption algorithm** used to securely encrypt and decrypt data. It was selected by the **U.S. National Institute of Standards and Technology (NIST)** in **2001** and is the global standard for protecting sensitive information.

> **Key idea:** The **same secret key** is used for both encryption and decryption.

### Simple Analogy

Think of AES as a **locked box**:

- You put your data inside the box.
- Lock it with a key.
- Anyone with the **same key** can unlock and read the data.

---

## Why it Matters

AES is trusted because it provides:

- **Confidentiality** – Keeps data hidden from unauthorized users.
- **High Security** – No practical attack exists against properly implemented AES.
- **Excellent Performance** – Fast enough for large files, databases, and network communication.
- **Global Standard** – Used by governments, banks, healthcare, and cloud providers.
- **Hardware Support** – Modern CPUs include AES instructions, making encryption extremely fast.

### Where AES is Used

- HTTPS/TLS (secure websites)
- VPNs
- Wi-Fi (WPA2/WPA3)
- Banking systems
- Password managers
- Cloud storage
- Mobile applications
- Disk encryption (BitLocker, FileVault)

---

## How it Works

### Step 1: Plain Text

Original readable data.

```
Hello World
```

---

### Step 2: Encryption Key

AES supports three key sizes:

|Key Size|Security Level|Rounds|
|---|---|---|
|128-bit|Strong|10|
|192-bit|Very Strong|12|
|256-bit|Extremely Strong|14|

Example:

```
Key = MySecretKey123
```

---

### Step 3: Encryption

AES performs several rounds of mathematical transformations:

- SubBytes
- ShiftRows
- MixColumns
- AddRoundKey

The readable text becomes unreadable.

```
Hello World

↓

4A 98 B3 F2 91 C7 ...
```

---

### Step 4: Transmission / Storage

The encrypted data (ciphertext) can safely travel across the internet or be stored in a database.

Even if someone intercepts it, they cannot understand it without the key.

---

### Step 5: Decryption

Using the **same key**, AES reverses the transformations.

```
Ciphertext

↓

Hello World
```

---

### AES Flow Diagram

```
Plain Text
     │
     ▼
Encryption
(using Secret Key)
     │
     ▼
Cipher Text
     │
     ▼
Decryption
(using Same Secret Key)
     │
     ▼
Original Plain Text
```

---

## Common Pitfalls

### 1. Hardcoding Keys

❌ Bad

```
String key = "mysecretpassword";
```

✔ Better

- Store keys in a secure vault.
- Use environment variables.
- Use cloud key management services.

---

### 2. Using ECB Mode

❌ Avoid

```
AES/ECB/PKCS5Padding
```

ECB encrypts identical data blocks into identical ciphertext blocks, revealing patterns.

✔ Prefer

- AES-GCM (recommended)
- AES-CBC with a random IV

---

### 3. Reusing Initialization Vectors (IVs)

Every encryption should use a **new random IV**.

Reusing IVs weakens security.

---

### 4. Weak Keys

Bad examples:

```
123456
password
abcdef
```

Use cryptographically secure random keys.

---

### 5. Forgetting Authentication

Encryption alone does not verify whether the data was modified.

Prefer **AES-GCM**, which provides:

- Encryption
- Integrity
- Authentication

---

## Spring Boot / Java Notes

### Maven Dependency

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

(Java Cryptography Extension is included in the JDK.)

---

### Generate an AES Key

```
KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
keyGenerator.init(256);
SecretKey key = keyGenerator.generateKey();
```

---

### Encrypt Data

```
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

cipher.init(Cipher.ENCRYPT_MODE, secretKey);

byte[] encrypted = cipher.doFinal(data.getBytes());
```

---

### Decrypt Data

```
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

cipher.init(Cipher.DECRYPT_MODE, secretKey, gcmSpec);

String text = new String(cipher.doFinal(encrypted));
```

---

### Best Practices

- ✅ Use **AES-256** when available.
- ✅ Prefer **AES-GCM** over ECB or CBC.
- ✅ Generate keys using `KeyGenerator`.
- ✅ Never hardcode keys.
- ✅ Store keys securely (AWS KMS, Azure Key Vault, HashiCorp Vault, etc.).
- ✅ Use a fresh random IV for every encryption.
- ✅ Rotate keys periodically.

---

## Interview Questions

### 1. What is AES?

AES is a **symmetric encryption algorithm** that uses the same secret key for encryption and decryption.

---

### 2. Why is AES called a symmetric algorithm?

Because the **same key** is used to encrypt and decrypt the data.

---

### 3. What key sizes does AES support?

- 128-bit
- 192-bit
- 256-bit

---

### 4. Which AES mode is recommended?

**AES-GCM** because it provides:

- Confidentiality
- Integrity
- Authentication

---

### 5. Why should ECB mode be avoided?

ECB reveals patterns in encrypted data because identical plaintext blocks produce identical ciphertext blocks.

---

### 6. What is an IV (Initialization Vector)?

An IV is a random value used along with the encryption key to ensure that encrypting the same plaintext multiple times produces different ciphertext.

---

### 7. Is AES enough for password storage?

**No.** Passwords should be **hashed**, not encrypted.

Use algorithms like:

- BCrypt
- SCrypt
- Argon2

---

### 8. Difference between AES and RSA?

|AES|RSA|
|---|---|
|Symmetric encryption|Asymmetric encryption|
|Same key for encrypt/decrypt|Public and private keys|
|Very fast|Slower|
|Encrypts large amounts of data|Mainly used for key exchange and digital signatures|

---

### 9. Why is AES faster than RSA?

AES performs efficient block-based symmetric encryption, while RSA relies on computationally expensive public-key mathematics.

---

### 10. Can AES be cracked?

With current technology, properly implemented AES-128, AES-192, and AES-256 are considered computationally infeasible to brute-force.

---

## References

- **NIST FIPS 197** – Advanced Encryption Standard (AES)
- **Java Cryptography Architecture (JCA)** Documentation
- **OWASP Cryptographic Storage Cheat Sheet**
- **Oracle Java Cipher API Documentation**

---

## Quick Revision (1-Minute Recap)

| Topic                                | Remember                                           |
| ------------------------------------ | -------------------------------------------------- |
| Type                                 | Symmetric Encryption                               |
| Standard                             | AES (Advanced Encryption Standard)                 |
| Key Sizes                            | 128, 192, 256 bits                                 |
| Block Size                           | 128 bits                                           |
| Best Mode                            | AES-GCM                                            |
| Avoid                                | AES-ECB                                            |
| Uses                                 | HTTPS, VPN, Wi-Fi, Cloud, Banking, Disk Encryption |
| Password Storage                     | ❌ No (use BCrypt/Argon2 instead)                   |
| Same Key for Encryption & Decryption | ✅ Yes                                              |
| Main Advantage                       | Fast, Secure, Widely Supported                     |