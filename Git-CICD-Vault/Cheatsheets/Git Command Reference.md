---
tags: [git, cheatsheet]
---
# Git Command Reference
Back to [[00 - Index (MOC)|Index]]

| Command | Purpose |
|---|---|
| `git init` | Initialize repository |
| `git status` | Check current changes |
| `git add .` | Stage changes |
| `git commit -m "message"` | Create commit |
| `git log --oneline` | View commit history |
| `git branch` | List branches |
| `git switch -c feature/name` | Create and switch branch |
| `git switch main` | Switch branch |
| `git merge feature/name` | Merge a branch |
| `git remote -v` | View remote repositories |
| `git push` | Upload commits |
| `git pull` | Download and integrate changes |
| `git fetch` | Download remote changes |
| `git diff` | Inspect unstaged changes |
| `git diff --staged` | Inspect staged changes |
| `git restore filename` | Discard unstaged changes |
| `git reset` | Unstage changes |
| `git stash` | Temporarily save working changes |

> [!danger]
> `git reset --hard` and `git clean` can **permanently discard local work**.
