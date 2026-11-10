---
tags: [git, lesson, hands-on]
lesson: 7
---
# Lesson 7 – Your First Commit
⬅️ [[06 - gitignore]] | ➡️ [[08 - Branches]]

```bash
git status          # sources + .gitignore untracked; target/ excluded
git add .           # move changes to staging area
git status
```
```text
Changes to be committed:
    new file: .gitignore
    new file: pom.xml
    new file: src/main/java/...
```
```bash
git commit -m "Initial Spring Boot application"
```
🎉 First commit!

## What happens internally
1. Calculates hashes for file contents
2. Stores contents as **blob** objects
3. Creates **tree** objects for directories
4. Creates a **commit** object (snapshot + metadata)
5. Updates the current branch reference

```bash
git log
git log --oneline
```
```text
a82f341 Initial Spring Boot application
```
`a82f341` is an abbreviated **commit hash** identifying the commit.
