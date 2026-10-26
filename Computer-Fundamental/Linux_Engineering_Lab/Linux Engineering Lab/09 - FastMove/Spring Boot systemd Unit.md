---
type: reference
status: planned
tags: [fastmove, systemd, spring-boot]
---

# Spring Boot systemd Unit (starter)

Adapt paths, user and Java location to your host. Check with `command -v java`.

## `/etc/systemd/system/fastmove.service`
```ini
[Unit]
Description=FastMove service
After=network-online.target
Wants=network-online.target

[Service]
User=fastmove
Group=fastmove
WorkingDirectory=/opt/fastmove
EnvironmentFile=-/etc/fastmove/fastmove.env
ExecStart=/usr/bin/java -Xms512m -Xmx1g -jar /opt/fastmove/app.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
LimitNOFILE=65536

# Hardening (verify each against your app before enabling)
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

## Why each choice
| Line | Reason |
|---|---|
| `User=fastmove` | Least privilege, [[Users and Groups]] |
| `WorkingDirectory` | Do not depend on cwd, [[Absolute vs Relative Paths]] |
| `EnvironmentFile=-...` | Secrets/config outside Git; leading `-` tolerates a missing file |
| `SuccessExitStatus=143` | JVM exits 143 on SIGTERM; treat it as clean stop |
| `TimeoutStopSec` | Give Spring time for graceful shutdown before SIGKILL |
| `LimitNOFILE` | Avoid "too many open files" under load |
| `Restart=on-failure` | Auto-recover from crashes without restart loops on clean stop |

## Permissions for the env file
```bash
sudo chown root:fastmove /etc/fastmove/fastmove.env
sudo chmod 640 /etc/fastmove/fastmove.env
```

## Install and verify
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now fastmove
systemctl status fastmove
journalctl -u fastmove -f
ss -ltnp | grep 8080
curl -s http://127.0.0.1:8080/actuator/health
```
Actuator may be disabled or secured, so adapt the health check.

## Break-it exercises
1. Wrong jar path → predict the status code.
2. Remove read permission on the env file → what does the journal say?
3. Start a second listener on the port → follow [[Lab 03 - Port Already in Use]].
4. Set `-Xmx` too high for the host → look for OOM evidence.
