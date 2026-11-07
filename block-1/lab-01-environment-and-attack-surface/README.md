# Lab 1 — Environment Setup and First Contact: the Attack Surface of a Machine

**Goal:** enumerate the local attack surface of the course lab VM (`ssc-lab`) using read-only commands only.

The full answers are in the [Block 1 report](../Report-Block1.pdf).

## What I did

I looked at the VM along four dimensions:

| Dimension | Question | Tools |
|---|---|---|
| What listens | Which ports accept connections, and as which user? | `ss -tlnp`, `ps`, `systemctl`, `curl`, `sshd -T` |
| What is privileged | Which binaries are setuid, and what does `sudo` allow? | `find / -perm -4000`, `ls -l`, `sudo -l` |
| What is scheduled | What runs automatically as root? | `ls -l /etc/cron.d/`, `cat` |
| What is exposed at rest | Which secrets can any user read? | `ls -l`, `cat` |

I also audited the SSH configuration: it is assembled from drop-ins read in lexical order, and only the daemon itself (`sshd -T`) reports the effective posture.

## Findings

A consolidated inventory of four focus items:

| # | Surface item | Dimension | Runs/owned as |
|---|---|---|---|
| 1 | `report.service` (`python3 /opt/report/server.py`, port 8081, bound to `0.0.0.0`, no authentication) | What listens | `svc-report` |
| 2 | `/usr/local/bin/backup-tool` (a copy of `cp` with the setuid bit) | What is privileged | root |
| 3 | `/etc/cron.d/ssc-maint` running the world-writable `/opt/maint/rotate.sh` every 10 minutes | What is scheduled | root |
| 4 | `/etc/report/report.conf`, a world-readable credentials file | What is at rest | root |

## Course material

- Handout: [`lab-01-environment-and-attack-surface.pdf`](lab-01-environment-and-attack-surface.pdf)
- Lecture: [`lecture-01-foundations-of-systems-security.pdf`](../lectures/lecture-01-foundations-of-systems-security.pdf)

Both were written by the course lecturer, who holds the rights to them.
