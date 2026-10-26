---
type: lab
status: planned
tags: [lab, permissions]
---

# Lab 02: Permissions Break and Fix

**Objective:** reproduce and diagnose "Permission denied" on a nested path using evidence.
**Safety:** only `/tmp/lab02`. Use a normal user; do not do this as root.

## Setup
```bash
mkdir -p /tmp/lab02/app/conf
echo "port=8080" > /tmp/lab02/app/conf/app.properties
```

## Break it (one at a time; predict each result)
1. `chmod 000 /tmp/lab02/app/conf/app.properties` then `cat` it.
2. Restore file to `644`. `chmod 600 /tmp/lab02/app/conf` (remove `x` from the directory) then `cat` the file again.
3. Restore. `chmod 700 /tmp/lab02/app` then test as another user if available: `sudo -u nobody cat /tmp/lab02/app/conf/app.properties`.

## Diagnose each with
```bash
namei -l /tmp/lab02/app/conf/app.properties
```
Which component blocked access each time?

> [!question]- Hint
> Step 2 fails even though the *file* is readable: no `x` on `conf` means no traversal. Step 3 fails at `app` for "others".

## Fix properly
Give the minimum: directory `755` (or `750` with right group), file `640` with correct group. Never `777`.

## Cleanup
`chmod -R u+rwX /tmp/lab02 && rm -rf /tmp/lab02`

## Completion test
Given a "Permission denied" on a deep path, name your first command and why.
