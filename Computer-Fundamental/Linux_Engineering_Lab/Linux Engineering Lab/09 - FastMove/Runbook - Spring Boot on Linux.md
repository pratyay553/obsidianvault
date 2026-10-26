---
type: runbook
status: planned
tags: [fastmove, spring-boot, deployment]
---

# Runbook: Spring Boot on Linux

## Goal
Run, verify and troubleshoot a FastMove service predictably.

## Pre-flight checklist
- [ ] `java -version` matches what the app requires
- [ ] JAR path, owner and version recorded
- [ ] Config source known (env file, `application.yml`, config server)
- [ ] Port and bind address decided
- [ ] DB / Kafka endpoints reachable from this host
- [ ] Log destination and retention decided
- [ ] Secrets in protected runtime config, **never** in this vault or Git
- [ ] Rollback plan written

## First-response diagnostics
```bash
systemctl status YOUR_UNIT
journalctl -u YOUR_UNIT --since "30 min ago" -p warning
ss -ltnp | grep java
free -h; df -hT; df -ih; uptime
ps -o pid,user,rss,etime,cmd -C java
```

## Failure classification
| Symptom | First checks |
|---|---|
| Won't start, bind error | [[Lab 03 - Port Already in Use]] |
| Exit 203/200/217 | Unit file paths and user, [[systemd Services]] |
| Permission denied | `namei -l`, `sudo -u` test, [[Permissions and Ownership]] |
| DB connection error | `getent hosts`, route, TCP connect, credentials config, DB logs |
| Kafka connection error | Bootstrap address, advertised listeners, DNS, network path |
| Exit 137 | OOM evidence, [[CPU and Memory]] |
| Slow responses | Separate CPU, GC, DB, downstream latency |
| Disk errors | [[Storage and Filesystems]] |
| Many CLOSE_WAIT | Connection leak, [[TCP Connection States]] |

## Post-deploy verification
- [ ] Unit active and stays active for several minutes
- [ ] Port listening on intended address
- [ ] Health endpoint OK from the client's network location
- [ ] DB and Kafka dependencies verified
- [ ] No startup errors in journal
- [ ] Memory / CPU / disk within expected bounds
- [ ] Rollback tested or at least rehearsed

## Related
[[Spring Boot systemd Unit]] · [[Troubleshooting Method]]
