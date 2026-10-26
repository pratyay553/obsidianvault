---
type: reference
tags: [reference, glossary]
---

# Glossary

| Term | Plain meaning |
|---|---|
| **Kernel** | The core of the OS. Owns hardware, memory, processes, files, network. Programs ask it for services via system calls. |
| **Userland** | Everything outside the kernel: shell, tools, your JVM. |
| **Shell** | A program that reads commands and starts other programs (bash, zsh). |
| **Process** | A running instance of a program with its own PID, memory, credentials, open files. |
| **PID / PPID** | Process ID / parent process ID. |
| **Daemon / service** | A long-running background process, often managed by systemd. |
| **Unit (systemd)** | A managed object: service, timer, socket, mount. Defined by a unit file. |
| **Inode** | Filesystem record holding a file's metadata and data location. The *name* lives in the directory, not the inode. |
| **Mount** | Attaching a filesystem to a directory in the tree. |
| **File descriptor (fd)** | A small integer a process uses to refer to an open file, socket, or pipe. 0 stdin, 1 stdout, 2 stderr. |
| **Signal** | A small asynchronous message sent to a process (SIGTERM, SIGKILL, SIGHUP). |
| **Exit status** | Number returned by a process. 0 = success by convention. |
| **Socket** | Kernel object for communication; TCP sockets are identified by IP and port. |
| **Listening socket** | A socket waiting for incoming connections. Not proof the app is healthy. |
| **Load average** | Average count of runnable plus uninterruptible-wait tasks. Not the same as CPU %. |
| **Page cache** | RAM used to cache file data. Counts as reclaimable "available" memory. |
| **OOM killer** | Kernel mechanism that kills processes when memory is exhausted. |
| **cgroup** | Kernel feature that limits and accounts resources for groups of processes (basis of container limits). |
| **ACL** | Extra per-user/group permission entries beyond basic owner/group/other. |
| **SELinux / AppArmor** | Mandatory access control layers that can deny actions even when mode bits allow them. |
| **Idempotent** | Running an operation twice has the same effect as once. |
