---
type: concept
status: new
level: 2
tags: [linux, bash, automation]
---

# Bash Fundamentals

## Mental model
A script is a list of commands. The shell expands variables and globs **before** running each command. Most script bugs come from expansion and unchecked failures.

## Essentials
```bash
name="fast move"
echo "$name"            # quoted: one argument
echo $name              # unquoted: split into two words
cmd; echo "exit=$?"     # exit status of previous command
[[ -f /etc/hosts ]] && echo exists
for f in *.log; do wc -l "$f"; done
```

## Safe skeleton
```bash
#!/usr/bin/env bash
set -Eeuo pipefail
trap 'echo "failed at line $LINENO" >&2' ERR

APP_DIR="/opt/fastmove"
JAR="$APP_DIR/app.jar"

[[ -f "$JAR" ]] || { echo "missing: $JAR" >&2; exit 1; }
echo "ok: $JAR"
```

| Option | Effect | Caveat |
|---|---|---|
| `-e` | Exit on command failure | Not applied in `if`, `||`, `&&` contexts; surprising in some cases |
| `-u` | Error on unset variable | Prevents `rm -rf "$UNSET/"` |
| `-o pipefail` | Pipeline fails if any stage fails | Know that `grep` returning 1 now fails the pipe |

## Idempotency
Run twice, same result. Use `mkdir -p`, `ln -sfn`, check-before-create, and make restarts and retries safe.

## Scheduling
- `cron`: simple time-based jobs; minimal environment (set PATH, use absolute paths, log output).
- systemd timers: dependency-aware, journal logging, better observability.

## Checklist
- Quote every variable unless splitting is intended.
- Validate inputs and required files.
- Use absolute paths in services and cron.
- Never log secrets.
- Test the failure path, not only the happy one.
- Run `shellcheck script.sh` if available.

## Common mistakes
- Unquoted variables with spaces.
- Assuming cron has your PATH.
- `cd` without `|| exit`.
- Parsing `ls` output.

## Active recall
Why quote `"$var"`?:: Prevents word splitting and globbing.
What does `set -u` do?:: Treats unset variables as errors.
What does `$?` hold?:: Exit status of the last command.
Why prefer systemd timers over cron?:: Journal logging, dependencies, status visibility.
What is idempotency?:: Repeated execution yields the same end state.

## My explanation
> Describe how a deploy script should behave when it fails halfway:
