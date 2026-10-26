---
type: lab
status: planned
tags: [lab, storage]
---

# Lab 04: Disk Full Mystery (deleted-but-open file)

**Objective:** reproduce the classic "df says full, du finds nothing".
**Safety:** use a small tmpfs or a disposable VM, never a production disk.

## Setup (needs sudo)
```bash
sudo mkdir -p /mnt/lab04
sudo mount -t tmpfs -o size=64m tmpfs /mnt/lab04
```

## Reproduce
```bash
sudo dd if=/dev/zero of=/mnt/lab04/big.log bs=1M count=50
sudo tail -f /mnt/lab04/big.log &        # keeps the file open
sudo rm /mnt/lab04/big.log
df -h /mnt/lab04
sudo du -sh /mnt/lab04
```
**Predict:** how much does `df` show used? How much does `du` show?

> [!question]- Answer
> `df` still shows about 50 MB used; `du` shows ~0. The inode is unlinked but the open descriptor keeps the blocks.

## Investigate
```bash
sudo lsof +L1 /mnt/lab04
```
Note the PID holding the deleted file.

## Resolve
Stop the holding process (`kill` the `tail`). Re-run `df`. Space returns.

## Real-world mapping
A Java app keeps writing to a log that someone `rm`'d. Proper fix: rotate with a copytruncate/reopen strategy, or restart the service, not repeated deletes.

## Cleanup
```bash
sudo umount /mnt/lab04 && sudo rmdir /mnt/lab04
```
