---
type: lab
status: planned
tags: [lab, systemd]
---

# Lab 05: Service That Will Not Start

**Objective:** read systemd failure codes and fix three different faults.
**Safety:** use a VM. Names below are examples.

## Base unit `/etc/systemd/system/labsvc.service`
```ini
[Unit]
Description=Lab service

[Service]
User=labuser
WorkingDirectory=/opt/labsvc
ExecStart=/usr/bin/python3 -m http.server 9090 --bind 127.0.0.1
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Create the user and dir first:
```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin labuser
sudo mkdir -p /opt/labsvc && sudo chown labuser: /opt/labsvc
```

## Faults to inject (one at a time, then `daemon-reload` and `start`)
1. Change `ExecStart` to `/usr/bin/pythonX`.
2. Change `WorkingDirectory` to `/opt/doesnotexist`.
3. Change `User=` to `nosuchuser`.

For each: read `systemctl status labsvc` and `journalctl -u labsvc -e`. Predict the status code first.

> [!question]- Expected codes
> 1: `203/EXEC`. 2: `200/CHDIR`. 3: `217/USER`.

## Completion test
Fix all three from the status output alone, without looking at this note.

## Cleanup
```bash
sudo systemctl disable --now labsvc; sudo rm /etc/systemd/system/labsvc.service
sudo systemctl daemon-reload; sudo userdel labuser; sudo rm -rf /opt/labsvc
```
