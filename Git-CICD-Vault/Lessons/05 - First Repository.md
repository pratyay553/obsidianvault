---
tags: [git, lesson, hands-on]
lesson: 5
---
# Lesson 5 – Create Your First Repository
⬅️ [[04 - Install and Configure Git]] | ➡️ [[06 - gitignore]]

Project path from IntelliJ:
```text
/home/etrans/Downloads/scheduller
```
```bash
cd /home/etrans/Downloads/scheduller
git init
```
```text
Initialized empty Git repository in /home/etrans/Downloads/scheduller/.git/
```

## What happened?
Git created a hidden `.git/` directory – the **heart of the repository**:
```text
scheduller/
    .git/
    src/
    pom.xml
    target/
```
It contains: configuration, commit history, branch references, Git objects, staging information.
```bash
ls -la
```

> [!important]
> Git is tracking the repo but has **not saved any version** yet.

## Check status
```bash
git status
```
```text
On branch master
Untracked files:
    pom.xml
    src/
    target/
```
*Untracked* = Git sees the files but isn't tracking them. Next: [[06 - gitignore]].
