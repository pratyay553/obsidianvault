---
tags: [git, lesson, remote]
lesson: 10
---
# Lesson 10 – fetch, pull, push
⬅️ [[09 - Connect to GitHub]] | ➡️ [[11 - Pull Requests]]

## git push
```bash
git push origin main
```
`LOCAL REPOSITORY ──► GITHUB`

## git fetch
Download remote changes **without** touching your working branch.
```bash
git fetch origin
```
`GITHUB ──► LOCAL REMOTE-TRACKING REFERENCES` (e.g. `origin/main`, `origin/feature/payment`). Nothing is merged automatically.

## git pull
```bash
git pull origin main
```
`git pull = git fetch + integration` (merge by default; can be configured to rebase).

| Command | Purpose |
|---|---|
| `git push` | Upload local commits |
| `git fetch` | Download remote changes without integrating |
| `git pull` | Download and integrate remote changes |

## Real-world example
```bash
git fetch origin
git log HEAD..origin/main --oneline   # inspect what changed
git pull origin main                  # integrate
```
