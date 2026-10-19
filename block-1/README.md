# Block 1 — Attack Surface and Threat Modelling

**Deliverable:** [SSC-Block1-Lab-Report-EN.pdf](SSC-Block1-Lab-Report-EN.pdf), the individual lab report covering Labs 1 and 2.

**Environment:** the course lab VM (*SSC lab base image* v1.0, Ubuntu 24.04 LTS, amd64, hostname `ssc-lab`), running isolated in VirtualBox. No exploitation was performed; every observation comes from read-only commands.

## Lab 1 — Environment Setup and First Contact: the Attack Surface of a Machine

I enumerated the local attack surface of the VM along four dimensions:

| Dimension | Question | Tools |
|---|---|---|
| What listens | Which ports accept connections, and as which user? | `ss -tlnp`, `ps`, `systemctl`, `curl`, `sshd -T` |
| What is privileged | Which binaries are setuid, and what does `sudo` allow? | `find / -perm -4000`, `ls -l`, `sudo -l` |
| What is scheduled | What runs automatically as root? | `ls -l /etc/cron.d/`, `cat` |
| What is exposed at rest | Which secrets can any user read? | `ls -l`, `cat` |

The lab also covers SSH configuration auditing: the configuration is assembled from drop-ins read in lexical order, and only the daemon itself (`sshd -T`) reports the effective posture.

**Result:** a consolidated inventory of four focus items:

| # | Surface item | Dimension | Runs/owned as |
|---|---|---|---|
| 1 | `report.service` (`python3 /opt/report/server.py`, port 8081, bound to `0.0.0.0`, no authentication) | What listens | `svc-report` |
| 2 | `/usr/local/bin/backup-tool` (a copy of `cp` with the setuid bit) | What is privileged | root |
| 3 | `/etc/cron.d/ssc-maint` running the world-writable `/opt/maint/rotate.sh` every 10 minutes | What is scheduled | root |
| 4 | `/etc/report/report.conf`, a world-readable credentials file | What is at rest | root |

## Lab 2 — Threat Modelling in Practice

I turned the inventory into a threat model of the same machine:

- **Attacker profile:** a foothold as the `svc-report` service account, which can also run `/usr/bin/find` as any user through `sudo` without a password.
- **Assets:** ranked by value to that attacker.
- **Data-flow diagram:** three named trust boundaries (B1 network, B2 root privilege, B3 service account).
- **STRIDE:** applied element by element to every process, data store, external entity and data flow, with the three most serious threats identified.
- **Attack tree:** for the goal *obtain root on the VM*, with a stated difficulty convention, deferred leaves and the cheapest attack set.
- **Prioritisation:** a ranked list of seven weaknesses with remediations, a reproducible attack-surface metric (current value 5, 4 after the top fix) and the policy line behind the top remediation.

## Key finding

`/opt/maint/rotate.sh` is world-writable and executed as root by cron every 10 minutes. An unprivileged account can therefore reach root with no exploit, no extra tools and no privileges beyond its own. This is the cheapest path in the attack tree and the top-ranked remediation.

## Report contents

1. Preparation questions (both labs)
2. Lab 1: Exercises 1–7 (orientation, listening sockets, SSH posture, setuid binaries, sudo, cron, readable configuration, inventory) and reflection
3. Lab 2: attacker and assets, DFD, STRIDE table, attack tree, ranked weaknesses, surface metric, policy line and reflection

> The credentials visible in some screenshots are fake values planted by the course lab image for educational purposes; they are not real secrets.
