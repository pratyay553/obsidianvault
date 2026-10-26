---
type: concept
status: new
level: 1
tags: [linux, users, security]
---

# Users and Groups

## Why it matters
Every process runs *as* a user. Permission errors are almost always "this user cannot do this to that object". You must know **which user** a service really runs as.

## Mental model
The kernel does not know names; it knows numbers: **UID** and **GID**. Names are looked up in `/etc/passwd`, `/etc/group` (and possibly LDAP or other sources). A process carries a UID, a primary GID, and supplementary groups. Access decisions use those numbers.

## Inspect
```bash
id                    # uid, gid, groups of current user
id someuser
getent passwd someuser
getent group docker
ps -o user,pid,cmd -C java
```

## System vs normal users
- UID 0 is root: bypasses most permission checks.
- Service accounts are usually non-login system users (no shell, no home needed) so a compromised service has few rights.

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin fastmove
```
Shell path varies by distro (`/sbin/nologin`, `/usr/sbin/nologin`).

## Predict, then observe
```bash
id
getent passwd root
```
**Predict:** what is root's UID, and what field is the login shell?

> [!question]- Answer
> UID 0. The shell is the last colon-separated field of the passwd line.

## Common mistakes
- Running services as root "because it works".
- Adding a user to a group and expecting the **current** shell to see it (new login or `newgrp` needed).
- Confusing the user who *deployed* with the user who *runs* the service.

## Active recall
What does the kernel use to identify users?:: Numeric UID and GID.
Why run services as a dedicated non-login user?:: Least privilege: limits damage if compromised.
Where are users defined?:: `/etc/passwd` (plus `/etc/group`, and other NSS sources).
Why doesn't a new group take effect immediately?:: Group membership is read at login/process start.

## My explanation
> Explain how a Spring Boot service's identity affects what files it can read:

## Next
→ [[Permissions and Ownership]]
