---
type: concept
status: new
level: 2
tags: [linux, systemd, services]
---

# systemd Services

## Why it matters
On most modern distros, deploying a backend means writing and debugging a unit file.

## Mental model
systemd reads **unit files** (declarative INI-like text), builds a dependency graph, starts processes in dependency order, tracks them with cgroups, restarts them per policy, and routes their stdout/stderr to the journal.

## Daily commands
```bash
systemctl status UNIT            # state, PID, recent log lines
systemctl cat UNIT               # the effective unit file(s)
systemctl show UNIT -p User -p ExecStart -p WorkingDirectory
sudo systemctl daemon-reload     # after editing unit files
sudo systemctl restart UNIT
sudo systemctl enable --now UNIT # start now and at boot
systemctl list-units --failed
journalctl -u UNIT -e
```

## Anatomy of a service unit
```ini
[Unit]
Description=Example service
After=network-online.target
Wants=network-online.target

[Service]
User=exampleuser
WorkingDirectory=/opt/example
ExecStart=/usr/bin/example --flag
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

| Section | Purpose |
|---|---|
| `[Unit]` | Description and ordering/dependencies |
| `[Service]` | How to run: user, paths, command, restart policy |
| `[Install]` | What `enable` hooks into at boot |

## Key distinctions
- `After=` is **ordering only**; `Wants=`/`Requires=` express dependency. You usually need both.
- `Restart=on-failure` restarts on non-zero exit or signal, not on clean stop.
- `Type=simple` (default) means "started" as soon as the process forks; it does not mean "ready". For readiness use `Type=notify` if the app supports it.
- Editing a unit without `daemon-reload` changes nothing.

## Reading `status`
```
Active: failed (Result: exit-code) since ...
Main PID: 4123 (code=exited, status=1/FAILURE)
```
`status=203/EXEC` means systemd could not execute `ExecStart` (bad path or permission). `status=200/CHDIR` means `WorkingDirectory` problem. `status=217/USER` means the `User=` does not exist. See [[Lab 05 - Service That Will Not Start]].

## Active recall
What must you run after editing a unit file?:: `systemctl daemon-reload`
Difference between `enable` and `start`?:: `enable` hooks into boot; `start` runs it now.
Is `After=` a dependency?:: No, only ordering.
What does 203/EXEC indicate?:: systemd failed to execute the `ExecStart` binary (path/permission/format).
How do you see the effective unit including drop-ins?:: `systemctl cat UNIT`
Where does service stdout go?:: The journal.

## My explanation
> Explain how systemd turns a unit file into a supervised process:

## Next
→ [[Logs and Journal]] · Template: [[Spring Boot systemd Unit]]
