---
type: concept
status: new
level: 1
tags: [linux, shell, navigation]
---

# Navigation and Inspection

## Why it matters
The first rule of operations: **look before you touch**. These commands let you see files, metadata and contents without modifying anything.

## Mental model
The shell always has a **current working directory (cwd)**. Most commands act relative to it. Inspection commands read; they should not change state. Know which is which.

## Core toolkit
| Command | Question it answers |
|---|---|
| `pwd` | Where am I? |
| `ls -lah` | What is here (including hidden), with sizes and owners? |
| `cd DIR`, `cd ..`, `cd -` | Move: into, up, back to previous |
| `file PATH` | What kind of thing is this really? |
| `stat PATH` | Exact metadata: size, inode, perms, owner, timestamps |
| `less FILE` | Read a big file safely (`/text` search, `q` quit, `F` follow) |
| `head -n 20`, `tail -n 50` | Start / end of a file |
| `tail -F FILE` | Follow a log, surviving rotation |
| `find DIR -name '*.log'` | Locate files by criteria |
| `which`, `type CMD` | What will actually run when I type this? |
| `wc -l`, `du -sh` | How many lines / how much space? |

## Predict, then observe
```bash
cd /tmp && mkdir -p nav-lab && cd nav-lab
printf 'one\ntwo\nthree\n' > a.txt
ls -l a.txt
stat a.txt
file a.txt
```
**Predict:** how many bytes is `a.txt`? (Count the newlines.)

> [!question]- Answer
> 14 bytes: `one\n` (4) + `two\n` (4) + `three\n` (6).

## `ls -l` vs `stat`
`ls -l` is a compact summary. `stat` shows the truth in full: inode number, link count, access/modify/change times, and exact permission bits. Use `stat` when "what exactly is this file?" matters.

## Safe habits
- Quote paths with spaces: `less "my logs/app.log"`.
- Preview before bulk changes: `find . -name '*.tmp' -print` **before** `-delete`.
- Run `pwd` before any destructive command.
- Avoid `rm -rf` until you can state exactly what the glob expands to. Test with `echo rm -rf ...` first.

> [!warning]
> `rm -rf $VAR/` with an empty `$VAR` becomes `rm -rf /`. Quote variables and use `set -u` in scripts.

## Common mistakes
- Using `cat` on a huge file (use `less`/`tail`).
- Using `tail -f` on rotated logs (use `-F`).
- Forgetting hidden files need `-a`.
- Assuming a file's extension tells you its type (use `file`).

## Active recall
What does `cd -` do?:: Returns to the previous directory.
Difference between `ls -l` and `stat`?:: `stat` shows full metadata (inode, link count, all timestamps, exact mode); `ls -l` is a summary.
Why `tail -F` instead of `-f`?:: `-F` re-opens the file by name, so it keeps following after log rotation or recreation.
How do you preview what `find ... -delete` would delete?:: Run the same `find` with `-print` first.

## My explanation
> Describe how you would safely inspect an unfamiliar directory before changing anything:

## Next
→ [[Absolute vs Relative Paths]]
