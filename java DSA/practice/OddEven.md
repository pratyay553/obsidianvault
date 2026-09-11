# Question: Odd or Even Number

## Problem

Write a Java program to determine whether a given number is **odd or even**.

### Example

**Input:**
```text
10
```

**Output:**
```text
Even
```

**Input:**
```text
7
```

**Output:**
```text
Odd
```

---

## My First Approach

> Write your solution before looking at the answer.

### My Thought Process

- How can I determine whether a number is odd or even?
- Which operator can help me?
- What should happen when the number is divisible by 2?
- What should happen when it is not divisible by 2?

### My Code

```java
// Write your solution here
```

---

## Syntax Used

### 1. Variable

```java
int number = 10;
```

### 2. `if-else`

```java
if (condition) {
    // code
} else {
    // code
}
```

### 3. Modulus Operator `%`

```java
number % 2
```

The `%` operator gives the **remainder** after division.

Example:

```text
10 % 2 = 0
11 % 2 = 1
```

Therefore:

```java
number % 2 == 0
```

means the number is divisible by 2.

---

## Internal Flow

### Java Program Execution

```text
Java Source Code
       ↓
     javac
       ↓
   Bytecode (.class)
       ↓
    ClassLoader
       ↓
      JVM
       ↓
 Interpreter / JIT
       ↓
 Machine Code
       ↓
      CPU
```

### What Happens?

1. We write Java source code in a `.java` file.
2. `javac` compiles the source code.
3. Java compiler produces bytecode.
4. The ClassLoader loads the bytecode into the JVM.
5. JVM executes the bytecode.
6. The JVM may interpret bytecode and/or use the JIT compiler to compile frequently executed code into native machine code.
7. CPU executes the resulting machine instructions.

> **Important:** For a tiny program like Odd/Even, don't imagine that every instruction necessarily goes through a simple "Interpreter → JIT → Machine Code" sequence. The JVM decides how code is executed and optimized at runtime.

---

## Solution

```java
public class OddEven {

    public static void main(String[] args) {

        int number = 10;

        if (number % 2 == 0) {
            System.out.println("Even");
        } else {
            System.out.println("Odd");
        }
    }
}
```

---

## How the Logic Works

Suppose:

```java
int number = 10;
```

Java evaluates:

```java
10 % 2
```

The result is:

```text
0
```

Then:

```java
0 == 0
```

is:

```text
true
```

Therefore:

```java
System.out.println("Even");
```

is executed.

For:

```java
int number = 7;
```

Java evaluates:

```text
7 % 2 = 1
```

So:

```java
1 == 0
```

is:

```text
false
```

Therefore the `else` block executes.

---

## Dry Run

### Input

```text
10
```

### Execution

```text
number = 10

number % 2
     ↓
10 % 2
     ↓
0

0 == 0
   ↓
 true
   ↓
Even
```

---

## Edge Cases

Test these:

| Input | Expected Output |
|---:|---|
| `0` | Even |
| `1` | Odd |
| `2` | Even |
| `-1` | Odd |
| `-2` | Even |
| `100` | Even |

---

## Time Complexity

```text
O(1)
```

We perform a constant number of operations.

## Space Complexity

```text
O(1)
```

We use only a fixed amount of memory.

---

## Common Mistakes

### Mistake 1: Using `/` instead of `%`

Wrong:

```java
if (number / 2 == 0)
```

Correct:

```java
if (number % 2 == 0)
```

---

### Mistake 2: Using assignment instead of comparison

Wrong:

```java
if (number % 2 = 0)
```

Correct:

```java
if (number % 2 == 0)
```

`=` means assignment.

`==` means comparison.

---

## Alternative Solution

You can also use the conditional operator:

```java
String result = number % 2 == 0 ? "Even" : "Odd";

System.out.println(result);
```

### Syntax

```java
condition ? valueIfTrue : valueIfFalse;
```

---

## Interview Explanation

> "To determine whether a number is odd or even, I use the modulus operator. If the remainder after dividing the number by 2 is zero, the number is even; otherwise, it is odd. The algorithm takes O(1) time and O(1) space."

---

## Interview Questions

### Q1. Why do we use `%` instead of `/`?

### Q2. What is the difference between `%` and `/`?

### Q3. What is the result of `10 % 2`?

### Q4. What is the result of `11 % 2`?

### Q5. Is `0` odd or even?

### Q6. Does the logic work for negative numbers?

### Q7. What is the time complexity?

### Q8. What is the space complexity?

### Q9. What happens internally when `number % 2` executes?

### Q10. What is the difference between `=` and `==`?

---

## Internal Java Concepts

This simple problem can teach:

- [[Java Source Code]]
- [[javac]]
- [[Bytecode]]
- [[JVM]]
- [[ClassLoader]]
- [[JIT Compiler]]
- [[Primitive Data Types]]
- [[int]]
- [[Variables]]
- [[if-else]]
- [[Operators]]
- [[Modulus Operator]]
- [[Boolean Expression]]
- [[CPU]]

---

## What I Learned

- A number is even when `number % 2 == 0`.
- `%` returns the remainder.
- `==` compares values.
- `if-else` controls program flow.
- Java source code is compiled into bytecode.
- JVM executes Java bytecode.
- Java can use JIT compilation to optimize frequently executed code.

---

## My Final Understanding

### In One Sentence

> **If a number leaves remainder `0` when divided by `2`, it is even; otherwise, it is odd.**

### Confidence

- [ ] I can write the solution without looking.
- [ ] I can explain `%`.
- [ ] I can explain `if-else`.
- [ ] I understand the Java execution flow.
- [ ] I can explain the solution in an interview.