---
type: index
tags: [home]
---

# Linux Engineering Lab

> Goal: understand Linux well enough to **operate, diagnose and deploy** backend services (Spring Boot, databases, Kafka) with confidence, and to explain *why* things happen, not just which command to type.

## How to use this vault
1. Read [[Study Method]] once. It is short and it is the engine of the whole vault.
2. Follow the [[Learning Path]] in order. Each note has the same shape, so you always know where you are.
3. Track yourself in [[Progress Tracker]].
4. Every day, create a note from `Templates/Template - Daily Log` (or use the Daily Notes button).
5. Every week, fill [[Weekly Review]].

## Map of content
| Area | Notes |
|---|---|
| Foundations | [[Filesystem Hierarchy]], [[Navigation and Inspection]], [[Absolute vs Relative Paths]], [[Inodes, Links and Everything-is-a-File]], [[Streams, Pipes and Redirection]] |
| Users & permissions | [[Users and Groups]], [[Permissions and Ownership]], [[sudo and Privilege]] |
| Processes & services | [[Processes and Signals]], [[systemd Services]] |
| Logs & troubleshooting | [[Logs and Journal]], [[Troubleshooting Method]] |
| Networking | [[Ports, DNS and Connectivity]], [[TCP Connection States]] |
| Resources | [[CPU and Memory]], [[Storage and Filesystems]] |
| Automation | [[Bash Fundamentals]] |
| Labs | [[Lab 01 - Explore the Filesystem]], [[Lab 02 - Permissions Break and Fix]], [[Lab 03 - Port Already in Use]], [[Lab 04 - Disk Full Mystery]], [[Lab 05 - Service That Will Not Start]] |
| FastMove | [[Runbook - Spring Boot on Linux]], [[Spring Boot systemd Unit]] |
| Reference | [[Command Cheat Sheet]], [[Glossary]], [[Flashcards - Linux Core]] |

## Rules of the lab
- Practise only in a VM, container, WSL, or a disposable machine. Never experiment on production.
- Redact secrets, tokens, hostnames, public IPs and customer data from notes.
- Evidence beats guesses: always write what an output **proves** and what it **does not prove**.
- Note the distro and version (`cat /etc/os-release`) when behaviour might differ.
