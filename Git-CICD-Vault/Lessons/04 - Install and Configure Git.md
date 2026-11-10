---
tags: [git, lesson, setup]
lesson: 4
---
# Lesson 4 – Install Git and Configure Identity
⬅️ [[03 - Git Architecture]] | ➡️ [[05 - First Repository]]

## Install (Ubuntu/Linux)
```bash
sudo apt update
sudo apt install git
git --version      # e.g. git version 2.43.0
```

## Configure identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --list
```

## Behind the scenes
Global config lives in `~/.gitconfig` (`~` = home directory):
```bash
cat ~/.gitconfig
```
```ini
[user]
    name = Your Name
    email = your-email@example.com
```
Git uses this as the **commit author identity**.

> [!warning]
> This identity is **not** GitHub authentication.
