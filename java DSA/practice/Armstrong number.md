# Question: Armstrong Number

## Problem

Write a Java program to check whether a given number is an **Armstrong number**.

### What is an Armstrong Number?

For a number with `n` digits, if the sum of each digit raised to the power `n` is equal to the original number, then the number is called an **Armstrong number**.

### Example 1

```text
153
```

`153` has 3 digits.

Therefore:

```text
1³ + 5³ + 3³
= 1 + 125 + 27
= 153
```

So:

```text
153 → Armstrong Number
```

### Example 2

```text
123
```

```text
1³ + 2³ + 3³
= 1 + 8 + 27
= 36
```

```text
36 != 123
```

Therefore:

```text
123 → Not an Armstrong Number
```

### Other Armstrong Numbers

```text
0
1
2
3
4
5
6
7
8
9
153
370
371
407
1634
8208
9474
```

---

# My First Approach

> Write your solution before looking at the solution.

### My Thought Process

I need to:

1. Take a number.
2. Find how many digits it contains.
3. Extract each digit.
4. Raise each digit to the power of the number of digits.
5. Add all the results.
6. Compare the final sum with the original number.

### Important Questions

Before coding, think about:

- How can I extract the last digit?
- How can I remove the last digit?
- How can I count the digits?
- How can I calculate a digit's power?
- How can I preserve the original number while modifying a copy?

### My Code

```java
// Write your solution here
```

---

# Syntax Used

## 1. Variable Declaration

```java
int number = 153;
```

We use `int` to store the number.

---

## 2. Assignment

```java
int temp = number;
```

`temp` receives a copy of the value stored in `number`.

---

## 3. While Loop

```java
while (temp != 0) {
    // code
}
```

The loop continues while `temp` is not zero.

We use it to process each digit.

---

## 4. Modulus Operator `%`

```java
int digit = temp % 10;
```

The `%` operator gives the remainder.

For example:

```text
153 % 10 = 3
```

So we can extract the last digit.

```text
153 → 3
```

---

## 5. Integer Division

```java
temp = temp / 10;
```

For integers, division removes the decimal part.

```text
153 / 10 = 15
15 / 10  = 1
1 / 10   = 0
```

Therefore, repeatedly dividing by `10` removes the last digit.

---

## 6. `Math.pow()`

```java
Math.pow(digit, digits)
```

This calculates:

```text
digit ^ digits
```

For example:

```java
Math.pow(5, 3)
```

produces:

```text
125
```

### Important

`Math.pow()` returns a `double`.

Therefore, if we want an integer result, we may cast it:

```java
(int) Math.pow(digit, digits)
```

---

## 7. `if-else`

```java
if (sum == number) {
    // Armstrong
} else {
    // Not Armstrong
}
```

`==` compares two values.

---

# Important Operators

| Operator | Meaning | Example |
|---|---|---|
| `%` | Remainder | `153 % 10 → 3` |
| `/` | Division | `153 / 10 → 15` |
| `==` | Equality | `sum == number` |
| `+=` | Add and assign | `sum += power` |
| `!=` | Not equal | `number != 0` |

---

# Internal Flow

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

## What Happens?

### 1. Java Source Code

We write:

```java
public class Armstrong {
    public static void main(String[] args) {
        // logic
    }
}
```

This is human-readable Java source code.

---

### 2. `javac`

The Java compiler:

```text
javac Armstrong.java
```

compiles the source code.

The result is bytecode:

```text
Armstrong.class
```

---

### 3. Bytecode

Bytecode is an intermediate representation designed to be executed by the JVM.

It is not directly CPU-specific machine code.

---

### 4. ClassLoader

The JVM's ClassLoader loads the required class into memory.

Conceptually:

```text
Armstrong.class
       ↓
   ClassLoader
       ↓
      JVM
```

---

### 5. JVM

The JVM executes the bytecode.

The JVM can interpret bytecode and use the **JIT compiler** to compile frequently executed code into native machine code.

---

### 6. CPU

Eventually, native machine instructions are executed by the CPU.

> **Important:** Don't think of every Java statement as always following a simple `Interpreter → JIT → Machine Code` sequence. The JVM dynamically decides how to execute and optimize code at runtime.

---

# Solution — Beginner Friendly

```java
public class Armstrong {

    public static void main(String[] args) {

        int number = 153;

        // Keep the original number safe
        int originalNumber = number;

        // Used to count digits
        int digits = 0;

        int temp = number;

        // Count the number of digits
        while (temp != 0) {
            digits++;
            temp = temp / 10;
        }

        // Reset temp
        temp = number;

        int sum = 0;

        // Extract each digit and calculate its power
        while (temp != 0) {

            int digit = temp % 10;

            sum += (int) Math.pow(digit, digits);

            temp = temp / 10;
        }

        // Compare calculated sum with original number
        if (sum == originalNumber) {
            System.out.println(originalNumber + " is an Armstrong number");
        } else {
            System.out.println(originalNumber + " is not an Armstrong number");
        }
    }
}
```

---

# How the Solution Works

Suppose:

```java
int number = 153;
```

We first count the digits.

```text
153
 ↓
15
 ↓
1
 ↓
0
```

Therefore:

```text
digits = 3
```

Now we process every digit.

---

# Dry Run

## Initial State

```text
number = 153
originalNumber = 153
digits = 0
temp = 153
```

---

## Step 1 — Count Digits

```text
temp = 153
digits = 1

temp = 15
digits = 2

temp = 1
digits = 3

temp = 0
STOP
```

Therefore:

```text
digits = 3
```

---

# Step 2 — Calculate Armstrong Sum

Reset:

```text
temp = 153
sum = 0
```

### First iteration

```text
temp = 153

digit = 153 % 10
      = 3
```

Calculate:

```text
3³ = 27
```

So:

```text
sum = 27
```

Remove the last digit:

```text
153 / 10 = 15
```

---

### Second iteration

```text
temp = 15

digit = 15 % 10
      = 5
```

Calculate:

```text
5³ = 125
```

Now:

```text
sum = 27 + 125
    = 152
```

Remove the last digit:

```text
15 / 10 = 1
```

---

### Third iteration

```text
temp = 1

digit = 1 % 10
      = 1
```

Calculate:

```text
1³ = 1
```

Now:

```text
sum = 152 + 1
    = 153
```

Remove the last digit:

```text
1 / 10 = 0
```

Loop ends.

---

# Final Comparison

```text
sum          = 153
originalNumber = 153
```

Therefore:

```text
sum == originalNumber
```

is:

```text
true
```

Output:

```text
153 is an Armstrong number
```

---

# The Most Important Digit Trick

For number problems, remember these two operations:

```java
digit = number % 10;
number = number / 10;
```

They work together.

### Extract last digit

```java
number % 10
```

### Remove last digit

```java
number / 10
```

Example:

```text
     153
      ↓
153 % 10 → 3
153 / 10 → 15

      15
      ↓
15 % 10 → 5
15 / 10 → 1

       1
       ↓
1 % 10 → 1
1 / 10 → 0
```

This pattern is extremely important for beginner-level number problems.

You will reuse it for:

- Reverse number
- Palindrome number
- Sum of digits
- Count digits
- Armstrong number
- Digit frequency
- Number manipulation

---

# Time Complexity

Let `d` be the number of digits.

We scan the digits twice:

```text
1. Count digits → O(d)
2. Calculate sum → O(d)
```

Therefore:

```text
O(d)
```

Since:

```text
d = log10(n)
```

we can also express it as:

```text
O(log n)
```

---

# Space Complexity

We use only a few integer variables:

```text
number
originalNumber
temp
digits
digit
sum
```

Therefore:

```text
O(1)
```

---

# Edge Cases

Test the following:

```text
0
1
9
10
153
370
371
407
123
100
1634
```

### Expected examples

```text
0     → Armstrong
1     → Armstrong
153   → Armstrong
370   → Armstrong
371   → Armstrong
407   → Armstrong
123   → Not Armstrong
100   → Not Armstrong
```

---

# Important Edge Case: `0`

Our current implementation needs special consideration for:

```java
number = 0;
```

Because:

```java
while (temp != 0)
```

will execute zero times.

For a beginner solution, handle it explicitly:

```java
if (number == 0) {
    System.out.println("0 is an Armstrong number");
    return;
}
```

Then continue with the normal algorithm.

---

# Improved Solution

```java
public class Armstrong {

    public static void main(String[] args) {

        int number = 153;

        if (isArmstrong(number)) {
            System.out.println(number + " is an Armstrong number");
        } else {
            System.out.println(number + " is not an Armstrong number");
        }
    }

    static boolean isArmstrong(int number) {

        if (number == 0) {
            return true;
        }

        int originalNumber = number;

        int digits = 0;
        int temp = number;

        // Count digits
        while (temp != 0) {
            digits++;
            temp /= 10;
        }

        temp = number;

        int sum = 0;

        // Calculate Armstrong sum
        while (temp != 0) {

            int digit = temp % 10;

            sum += (int) Math.pow(digit, digits);

            temp /= 10;
        }

        return sum == originalNumber;
    }
}
```

---

# Alternative: Using `String`

Another approach is to convert the number to a String.

```java
int number = 153;

String value = String.valueOf(number);

int digits = value.length();
```

Then process each character.

This approach is easier to understand for some problems, but the arithmetic approach is more useful for learning **number manipulation**.

---

# Common Mistakes

## Mistake 1 — Losing the Original Number

Wrong:

```java
while (number != 0) {
    number /= 10;
}

if (sum == number) {
    // ...
}
```

After the loop:

```text
number = 0
```

So you can no longer compare against the original number.

Use:

```java
int originalNumber = number;
int temp = number;
```

---

## Mistake 2 — Forgetting to Reset `temp`

After counting digits:

```java
while (temp != 0) {
    ...
}
```

`temp` becomes:

```text
0
```

Therefore, before processing the digits again:

```java
temp = number;
```

---

## Mistake 3 — Using `/` to Extract the Digit

Wrong:

```java
int digit = number / 10;
```

This removes the last digit.

Correct:

```java
int digit = number % 10;
```

This extracts the last digit.

---

## Mistake 4 — Forgetting That `Math.pow()` Returns `double`

```java
Math.pow(5, 3)
```

returns a `double`.

If using an integer sum:

```java
sum += (int) Math.pow(digit, digits);
```

---

# Interview Explanation

> "To check whether a number is an Armstrong number, I first count the number of digits. Then I extract each digit using the modulus operator `% 10`. I raise each digit to the power of the total number of digits and add the results. Finally, I compare the calculated sum with the original number. If they are equal, the number is an Armstrong number. The time complexity is O(log n), where n is the number, and the auxiliary space is O(1)."

---

# Interview Questions

### Q1. What is an Armstrong number?

### Q2. Why do we use `% 10`?

### Q3. Why do we use `/ 10`?

### Q4. Why do we need `originalNumber`?

### Q5. Why do we need `temp`?

### Q6. Why do we count the number of digits first?

### Q7. What happens when the input is `0`?

### Q8. What is the time complexity?

### Q9. What is the space complexity?

### Q10. Why does `Math.pow()` return `double`?

### Q11. Can you solve Armstrong number without `Math.pow()`?

### Q12. Can you solve it without converting the number to a String?

### Q13. What happens internally when `temp % 10` is executed?

### Q14. What happens internally when `temp /= 10` is executed?

### Q15. How would you modify the program to find all Armstrong numbers between `1` and `100000`?

---

# Challenge

Try these **without looking at the solution**:

### Challenge 1

Create:

```java
static boolean isArmstrong(int number)
```

---

### Challenge 2

Print all Armstrong numbers from:

```text
1 → 10000
```

---

### Challenge 3

Find the Armstrong numbers between:

```text
100 → 1000000
```

---

### Challenge 4 — Advanced

Solve the problem **without using `Math.pow()`**.

Hint:

```text
5³
↓
5 × 5 × 5
```

Think about how you can calculate the power using a loop.

---

# Related Concepts

- [[Java Variables]]
- [[Primitive Data Types]]
- [[int]]
- [[Operators]]
- [[Modulus Operator]]
- [[Integer Division]]
- [[if-else]]
- [[while Loop]]
- [[Methods]]
- [[return]]
- [[Math.pow]]
- [[Time Complexity]]
- [[Space Complexity]]
- [[JVM]]
- [[Bytecode]]
- [[ClassLoader]]
- [[JIT Compiler]]

---

# What I Learned

- An Armstrong number compares a number with the sum of its powered digits.
- `% 10` extracts the last digit.
- `/ 10` removes the last digit.
- A temporary variable allows us to process the number without destroying the original.
- We need the number of digits to determine the power.
- `Math.pow()` returns `double`.
- Number-manipulation problems commonly use the `% 10` and `/ 10` pattern.
- The algorithm takes `O(log n)` time and `O(1)` extra space.

---

# My Final Understanding

```text
Number
   ↓
Count Digits
   ↓
Extract Last Digit (% 10)
   ↓
Raise Digit to Digit-Count Power
   ↓
Add to Sum
   ↓
Remove Last Digit (/ 10)
   ↓
Repeat
   ↓
Compare Sum with Original Number
   ↓
Armstrong / Not Armstrong
```

### One-Sentence Explanation

> **An Armstrong number is a number whose value equals the sum of each of its digits raised to the power of the total number of digits.**