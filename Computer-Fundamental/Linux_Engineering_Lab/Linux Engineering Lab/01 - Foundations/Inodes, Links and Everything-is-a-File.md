---
type: concept
status: new
level: 2
tags: [linux, filesystem, inode]
---

# Inodes, Links and Everything-is-a-File

## Why it matters
It explains deleted-but-still-open files filling disks, why `mv` is instant within a filesystem, why hard links share data, and why "inodes full" breaks a machine with free space.

## Mental model
A file has two separate parts:
- **Inode**: metadata (owner, mode, size, timestamps) plus pointers to data blocks. **No name.**
- **Directory entry**: a `name → inode number` mapping.

A "file name" is just a *link* to an inode. The inode keeps a **link count**. Data is freed only when the link count is 0 **and** no process still has it open.

```mermaid
flowchart LR
  D1["dir entry: report.txt"] --> I["inode 1234 (data, owner, mode)"]
  D2["dir entry: copy-link"] --> I
  S["symlink: shortcut"] -. stores path text .-> D1
```

## Link types
| Type | What it is | Survives target deletion? | Crosses filesystems? |
|---|---|---|---|
| Hard link (`ln a b`) | Another name for same inode | Yes, data lives while any name exists | No |
| Symbolic link (`ln -s a b`) | Small file holding a path | No, becomes dangling | Yes |

## "Everything is a file"
Many kernel objects are accessed with file operations (open/read/write): regular files, directories, devices (`/dev/null`), pipes, sockets, and virtual entries in `/proc`. Not everything is an ordinary disk file; the interface is what is uniform.

## Predict, then observe
```bash
cd /tmp && mkdir -p ino && cd ino
echo hello > a
ln a b            # hard link
ln -s a c         # symlink
ls -li
rm a
cat b
cat c
```
**Predict:** which of `b` and `c` still works after `rm a`?

> [!question]- Answer
> `b` works (the inode still has a name). `c` is dangling because it stores the *path* `a`, which no longer exists.

## The deleted-but-open file
```bash
sudo lsof +L1        # open files with link count 0
```
A process holding a deleted 10 GB log keeps the space allocated. `df` stays full while `du` shows nothing. Fix by restarting or signalling the process to reopen logs, not by deleting more files. See [[Lab 04 - Disk Full Mystery]].

## Inode exhaustion
```bash
df -ih
```
Millions of tiny files can exhaust inodes while `df -h` shows free space.

## Active recall
What does a directory entry contain?:: A name and an inode number.
When is file data actually freed?:: Link count is 0 and no process has it open.
Why can't hard links cross filesystems?:: Inode numbers are only meaningful inside one filesystem.
Which command shows inode usage?:: `df -ih`
Why can `df` show full while `du` shows little?:: Deleted files still held open by a process, among other causes.

## My explanation
> Explain name vs inode and why deleting a file may not free space:

## Next
→ [[Users and Groups]]
