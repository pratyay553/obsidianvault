# ABAC (Attribute-Based Access Control) – Simple Interview Notes

## What is ABAC?

**ABAC (Attribute-Based Access Control)** is an authorization model where access is decided by checking different **attributes (properties)** instead of assigning fixed roles.

In simple words:

> **"ABAC checks who you are, what you want to access, what you want to do, and under what conditions before allowing access."**

Unlike RBAC, there are **no fixed roles** controlling access. Every request is evaluated in real time.

---

# What are Attributes?

An **attribute** is simply a piece of information about the user, resource, action, or environment.

ABAC uses four types of attributes.

## 1. Subject Attributes (User Attributes)

These describe the **person requesting access**.

Examples:

- Department = Finance
    
- Job Title = Manager
    
- Clearance Level = Level 3
    
- Employee ID
    
- Employment Status
    

**Example**

User:

- Name = Rahul
    
- Department = Finance
    
- Clearance = Level 3
    

These values become the subject attributes.

---

## 2. Resource Attributes

These describe **what is being accessed**.

Examples:

- File Owner
    
- Department
    
- Project ID
    
- Classification = Confidential
    
- File Type
    

**Example**

Document:

- Department = Finance
    
- Classification = Confidential
    

These become the resource attributes.

---

## 3. Action Attributes

These describe **what operation the user wants to perform**.

Examples:

- Read
    
- Write
    
- Delete
    
- Approve
    
- Download
    
- Upload
    

**Example**

Rahul wants to **Read** a finance report.

Action = Read

---

## 4. Environment Attributes

These describe the **current situation or context**.

Examples:

- Time
    
- Location
    
- IP Address
    
- Device Type
    
- Network
    
- Day of Week
    

**Example**

- Time = 10:00 AM
    
- Device = Company Laptop
    
- Location = Office
    

These are environment attributes.

---

# How Does ABAC Work?

When a user sends a request, ABAC checks all these attributes against a policy.

### Example Policy

```
Allow access if:

User Department = Document Department
AND
User Clearance >= Document Classification
AND
Time is between 9 AM and 6 PM
AND
Device is Company Laptop
```

If **all conditions are true**, access is granted.

If **any condition fails**, access is denied.

---

# Step-by-Step Example

### User

- Department = Finance
    
- Clearance = Level 3
    

### Resource

- Department = Finance
    
- Classification = Level 2
    

### Action

- Read
    

### Environment

- Time = 11 AM
    
- Device = Company Laptop
    

Now ABAC checks:

✅ Same department? → Yes

✅ Clearance high enough? → Yes

✅ Business hours? → Yes

✅ Company device? → Yes

**Result: Access Allowed**

---

Now suppose Rahul tries again at **11 PM**.

Everything else is the same.

Time check:

❌ Business hours? → No

**Result: Access Denied**

Notice that the user didn't change—only the context changed.

---

# Real-Life Example

Imagine a company with a Finance folder.

Policy:

> "Finance employees can access Finance documents only during office hours and only from a company-managed laptop."

If Rahul:

- Works in Finance ✅
    
- Uses company laptop ✅
    
- Accesses during office hours ✅
    

Access is allowed.

If Rahul:

- Uses his personal laptop ❌
    

Access is denied.

If Rahul:

- Accesses at midnight ❌
    

Access is denied.

---

# Why Not Use RBAC Here?

In RBAC, you might need many different roles:

- Finance Employee
    
- Finance Employee Morning Shift
    
- Finance Employee Company Laptop
    
- Finance Employee Remote
    
- Finance Manager
    

As more conditions are added, the number of roles grows quickly.

This is called **Role Explosion**.

ABAC avoids this by checking attributes dynamically instead of creating many roles.

---

# Advantages of ABAC

### 1. Very Flexible

Access rules can include:

- Department
    
- Time
    
- Device
    
- Location
    
- Project
    
- Clearance
    

All together.

---

### 2. Fine-Grained Access Control

Different users can have different permissions even if they have the same job.

---

### 3. Context-Aware

Access can change depending on:

- Time
    
- Device
    
- Location
    
- Network
    

---

### 4. Fewer Roles Needed

No need to create hundreds of roles.

Policies handle access dynamically.

---

# Security Issues in ABAC

## 1. Policy Complexity

As more rules are added, policies become difficult to understand and maintain.

Example:

```
Finance
AND
Manager
AND
Office
AND
VPN
AND
Weekday
AND
Business Hours
AND
Company Device
```

Finding why access was denied can become difficult.

---

## 2. Attribute Integrity (Very Important)

ABAC completely depends on attribute values.

If an attacker changes an attribute, they may gain unauthorized access.

### Example

A JWT token contains:

```
department = Finance
```

If the attacker modifies it to:

```
department = Admin
```

and the server trusts it without verification, access may be incorrectly granted.

**Lesson:** Never trust client-supplied attributes without verification.

---

## 3. Performance Overhead

Every request requires checking multiple attributes and policies.

This is slower than RBAC, where the system often just checks a user's role.

---

## 4. Policy Conflicts

Example:

Policy 1:

```
Managers can view reports.
```

Policy 2:

```
Confidential reports cannot be viewed outside office hours.
```

A manager tries to view a confidential report at 10 PM.

One policy allows it, another denies it.

The system needs a clear conflict rule.

A common approach is:

> **Deny overrides Allow** (if any policy denies access, the final decision is Deny).

---

# How to Prevent These Issues

- Store user attributes securely on the server.
    
- Verify JWT claims using digital signatures.
    
- Never trust attributes sent directly by the client.
    
- Keep all authorization policies centralized.
    
- Regularly test and review policies.
    
- Use a clear conflict resolution strategy (commonly **Deny overrides Allow**).
    
- Cache attribute lookups carefully so data stays fresh while improving performance.
    

---

# RBAC vs ABAC

|RBAC|ABAC|
|---|---|
|Access based on **roles**|Access based on **attributes**|
|Static permissions|Dynamic permissions|
|Easier to implement|More flexible|
|Faster|Slightly slower due to policy evaluation|
|Can suffer from role explosion|Avoids role explosion|
|Best for simple systems|Best for complex, context-aware systems|

---

# Easy Interview Explanation

> **"ABAC (Attribute-Based Access Control) makes access decisions by evaluating attributes of the user, resource, action, and environment. Instead of assigning permissions through fixed roles, every request is checked against policies in real time. This makes ABAC highly flexible and suitable for context-aware access control, but it depends on trustworthy attribute data and well-managed policies."**

---

# 30-Second Interview Answer

> **"ABAC stands for Attribute-Based Access Control. It grants or denies access by checking attributes such as the user's department, the resource being accessed, the requested action, and environmental conditions like time or device. Unlike RBAC, which relies on predefined roles, ABAC evaluates policies dynamically for every request. It provides fine-grained, context-aware authorization, but requires secure attribute sources and carefully managed policies."