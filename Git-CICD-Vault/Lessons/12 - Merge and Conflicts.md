---
tags: [git, lesson, merge]
lesson: 12
---
# Lesson 12 – Merge and Merge Conflicts
⬅️ [[11 - Pull Requests]] | ➡️ [[13 - CI-CD Concepts]]

`main` contains:
```java
public class Employee {
    private String name;
}
```
You and another developer modify the same section on different branches. Git can't auto-combine → **merge conflict**.

```text
<<<<<<< HEAD
private String firstName;
=======
private String employeeName;
>>>>>>> feature/employee-management
```
These markers show the conflicting changes. **You decide** the final code:
```java
public class Employee {
    private String employeeName;
}
```
Then:
```bash
git add .
git commit -m "Resolve employee merge conflict"
```
The exact command depends on whether you're merging or rebasing, but the principle is the same.

> [!note]
> A merge conflict isn't necessarily a Git error. It means Git needs **your help** to decide how to combine changes.
