---
type: lab
status: planned
tags: [lab, filesystem]
---

# Lab 01: Explore the Filesystem

**Objective:** see mounts, virtual filesystems and metadata with your own eyes.
**Safety:** read-only except under `/tmp`.

## Steps
1. `findmnt -t ext4,xfs,btrfs,overlay,tmpfs,proc,sysfs` Which are real disks? Which are virtual?
2. `df -hT` Record the filesystem type and free space of `/`.
3. `ls -l /proc/self/cwd /proc/self/exe` What are these, and who is "self"?
4. `cat /proc/meminfo | head -5` Where does this data come from?
5. `mkdir /tmp/lab01 && cd /tmp/lab01 && echo hi > f && stat f` Identify inode, link count, size.
6. `ln f g && stat f` What changed? Predict first.

## Questions
- Why does `/proc/meminfo` have size 0 but contain text?
- What would `ln -s` have done differently in step 6?

> [!question]- Answers
> 3: `self` is the process reading it (here `ls`). 4: The kernel generates it when read, no disk involved. Step 6: link count becomes 2; same inode. A symlink would create a new inode holding a path.

## Cleanup
`rm -rf /tmp/lab01`

## Completion test
Explain, without notes, why `/proc` uses no disk space.
