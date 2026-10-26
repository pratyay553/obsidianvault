---
type: concept
status: new
level: 2
tags: [linux, sudo, security]
---

# sudo and Privilege

## Mental model
`sudo` runs **one command** as another user (usually root) after checking policy in `/etc/sudoers` (edit only with `visudo`). It is a privilege *elevator*, not a fixer: it does not make a wrong command correct.

## Key facts
- Redirections happen in your shell, not under sudo ([[Streams, Pipes and Redirection]]).
- Environment and PATH may be reset by policy.
- `sudo -u fastmove COMMAND` is the best way to test "can the service user do this?"
- Prefer targeted sudo rules over blanket `ALL`.
- Prefer capabilities or socket activation over running as root (e.g. binding ports below 1024).

```bash
sudo -u fastmove cat /etc/fastmove/app.env     # test as the service user
sudo -l                                         # what may I run?
```

## Common mistakes
- Running a build as root, then wondering why files are root-owned.
- `sudo` on commands you do not understand.
- Using `sudo` to "make it work" and never finding the real permission fault.

## Active recall
Why test with `sudo -u serviceuser`?:: It reproduces the real permissions the service has.
Safe way to edit sudoers?:: `visudo` (validates syntax).
Does sudo apply to shell redirection?:: No. Use `tee`.

## My explanation
> When is `sudo` the right tool and when is it hiding a problem?

## Next
→ [[Processes and Signals]]
