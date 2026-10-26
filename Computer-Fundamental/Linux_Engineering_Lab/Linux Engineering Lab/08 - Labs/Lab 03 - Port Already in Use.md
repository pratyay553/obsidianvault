---
type: lab
status: planned
tags: [lab, networking, spring-boot]
---

# Lab 03: Port Already in Use

**Symptom:** Spring Boot fails with "Address already in use" on 8080.

## Reproduce safely
Terminal A:
```bash
python3 -m http.server 8080 --bind 127.0.0.1
```
Terminal B: try to start another listener on 8080 (e.g. the same command) and read the error.

## Hypotheses
1. Another instance of the same app.
2. A different application.
3. Config points at a different port than assumed.

## Investigate
```bash
sudo ss -ltnp 'sport = :8080'
ps -fp PID_FROM_ABOVE
systemctl status UNIT      # if it is a service
```
Record: PID, user, command line, parent, how it was started.

## Resolve
Stop the **identified** process via its proper mechanism (Ctrl+C in A, or `systemctl stop`). If it is legitimate, give the new app a different port.

## Verify
- Second listener starts.
- `ss -ltnp` shows the expected owner.
- Health endpoint answers.

## Lesson
A bind error is a socket ownership problem, not a firewall problem. Identify before killing.

## Cleanup
Stop both servers.
