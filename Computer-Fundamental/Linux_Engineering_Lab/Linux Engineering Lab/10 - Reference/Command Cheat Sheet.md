---
type: reference
tags: [reference, commands]
---

# Command Cheat Sheet

Organised by **question**, not alphabet.

## Where am I / what is this?
| Question | Command |
|---|---|
| Current dir | `pwd` |
| Distro/version | `cat /etc/os-release` |
| What kind of file | `file X` |
| Full metadata | `stat X` |
| What will run | `type CMD` / `command -v CMD` |
| Mount tree | `findmnt` |

## Permissions
| Question | Command |
|---|---|
| Who am I | `id` |
| Perms on every path part | `namei -l PATH` |
| ACLs | `getfacl X` |
| SELinux label | `ls -Z X` |

## Processes
| Question | Command |
|---|---|
| Find process | `pgrep -af NAME` |
| Tree | `pstree -ap` |
| Top consumers | `ps -eo pid,user,%cpu,%mem,rss,cmd --sort=-%mem \| head` |
| Open files of process | `ls /proc/PID/fd` / `lsof -p PID` |
| Stop politely | `kill -TERM PID` |

## Services and logs
| Question | Command |
|---|---|
| State | `systemctl status UNIT` |
| Effective config | `systemctl cat UNIT` |
| Failed units | `systemctl --failed` |
| Logs | `journalctl -u UNIT -e` |
| Kernel/OOM | `journalctl -k` |

## Network
| Question | Command |
|---|---|
| Listening ports | `ss -ltnp` |
| Who owns port | `sudo ss -ltnp 'sport = :PORT'` |
| DNS as apps see it | `getent hosts NAME` |
| Routes | `ip route` |
| HTTP check | `curl -v --connect-timeout 3 URL` |

## Resources
| Question | Command |
|---|---|
| Load/cores | `uptime; nproc` |
| Memory | `free -h` |
| Disk space / inodes | `df -hT; df -ih` |
| Big dirs | `du -xhd1 PATH \| sort -h` |
| Deleted-open files | `sudo lsof +L1` |
| I/O stats | `iostat -x 1 3` |
