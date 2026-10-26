---
type: index
tags: [home, plan]
---

# Learning Path

Follow in order. Each phase ends with a **gate**: do not move on until you pass it.

## Phase 1: Orient yourself (week 1)
1. [[Filesystem Hierarchy]]
2. [[Navigation and Inspection]]
3. [[Absolute vs Relative Paths]]
4. [[Streams, Pipes and Redirection]]
5. Lab: [[Lab 01 - Explore the Filesystem]]

**Gate:** From memory, explain what `/etc`, `/var`, `/proc`, `/run` are for, and show a file's type, size, and owner using two different commands.

## Phase 2: Who can do what (week 2)
1. [[Inodes, Links and Everything-is-a-File]]
2. [[Users and Groups]]
3. [[Permissions and Ownership]]
4. [[sudo and Privilege]]
5. Lab: [[Lab 02 - Permissions Break and Fix]]

**Gate:** Diagnose "Permission denied" on a nested path using `namei -l` without guessing.

## Phase 3: Running things (week 3)
1. [[Processes and Signals]]
2. [[systemd Services]]
3. [[Logs and Journal]]
4. Labs: [[Lab 03 - Port Already in Use]], [[Lab 05 - Service That Will Not Start]]

**Gate:** Take a service that fails to start and find the first causal error in the journal.

## Phase 4: Systems under pressure (week 4)
1. [[CPU and Memory]]
2. [[Storage and Filesystems]]
3. [[Ports, DNS and Connectivity]]
4. [[TCP Connection States]]
5. [[Troubleshooting Method]]
6. Lab: [[Lab 04 - Disk Full Mystery]]

**Gate:** Explain why `df` and `du` can disagree and demonstrate it.

## Phase 5: Automate and ship (week 5+)
1. [[Bash Fundamentals]]
2. [[Spring Boot systemd Unit]]
3. [[Runbook - Spring Boot on Linux]]

**Gate:** Deploy a sample JAR as a systemd service, break it three different ways, and fix each using only logs and status output.
