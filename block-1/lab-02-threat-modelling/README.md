# Lab 2 — Threat Modelling in Practice

**Goal:** turn the Lab 1 inventory into a threat model of the same machine.

The full answers are in the [Block 1 report](../SSC-Block1-Lab-Report-EN.pdf).

## What I did

- **Attacker profile:** a foothold as the `svc-report` service account, which can also run `/usr/bin/find` as any user through `sudo` without a password.
- **Assets:** ranked by value to that attacker.
- **Data-flow diagram:** three named trust boundaries (B1 network, B2 root privilege, B3 service account).
- **STRIDE:** applied element by element to every process, data store, external entity and data flow, with the three most serious threats identified.
- **Attack tree:** for the goal *obtain root on the VM*, with a stated difficulty convention, deferred leaves and the cheapest attack set.
- **Prioritisation:** a ranked list of seven weaknesses with remediations, a reproducible attack-surface metric (current value 5, 4 after the top fix) and the policy line behind the top remediation.

## Key finding

`/opt/maint/rotate.sh` is world-writable and executed as root by cron every 10 minutes. An unprivileged account can therefore reach root with no exploit, no extra tools and no privileges beyond its own. This is the cheapest path in the attack tree and the top-ranked remediation.

## Course material

Handout: `lab-02-threat-modelling.pdf`. Lecture: `lecture-02-threat-modelling-in-practice.pdf`. Both are the lecturer's material and are not redistributed in this repository.
