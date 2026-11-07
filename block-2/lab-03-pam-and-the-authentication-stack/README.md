# Lab 3 — PAM and the Authentication Stack

**Goal:** follow a login from the process that runs it down to the module that checks the password, then break the stack on purpose, repair it and harden it.

The full answers are in the [Block 2 report](../SSC-Block2-Lab-Report-EN.pdf), Section 1.

## What I did

| Part | Exercises | Summary |
|---|---|---|
| A. Who is a process? Credentials and NSS | 1a, 2a, 3a, 3b | Read the four UIDs of a process; traced account lookups through `nsswitch.conf`; explained why `/etc/passwd` is world-readable and `/etc/shadow` is not, and which offline attack that split prevents |
| B. The PAM stack | 4a–4d | Read `/etc/pam.d/sudo` and `common-auth`; compared `required`, `sufficient` and `[success=1 default=ignore]` |
| C. Break it, then repair it | 5a, 5b | Added an unknown module to the `sudo` stack (`Module is unknown`), confirmed that PAM fails closed, and restored the file from a root terminal |
| D. Hardening the stack | 6a, 7a | Wrapped `pam_unix.so` with `pam_faillock` (`deny=3`, `unlock_time=120`); enforced password quality with `pam_pwquality` (`minlen = 12`, `dcredit = -1`, `ucredit = -1`) |

## Key commands

```bash
id
grep -E '^(Uid|Gid):' /proc/self/status        # real, effective, saved and filesystem IDs
grep -E '^(passwd|group|shadow):' /etc/nsswitch.conf
getent passwd svc-report
sudo getent shadow student | cut -d: -f1,2     # the hash field
cat /etc/pam.d/sudo /etc/pam.d/common-auth
faillock --user student                        # failure count after the lockout test
faillock --user student --reset
```

## Findings

- **PAM fails closed.** When a `required` module cannot be loaded, authentication is denied. A broken `sudo` stack costs availability, while a stack that failed open would hand out root.
- **The experiment was safe because of how it was set up:** only `/etc/pam.d/sudo` was changed (not `common-auth`), a root terminal stayed open and a snapshot existed.
- **Lockout is a trade-off.** `pam_faillock` cuts online guessing to about 90 attempts per hour, but anyone who knows a username can keep that account locked.
- **Password quality belongs to the `password` type**, because it gates the choice of a new credential, not a login attempt.

## Course material

Handout: `lab-03-pam-and-the-authentication-stack.pdf`. Lecture: `lecture-03-authentication-nss-and-pam.pdf`. Both are the lecturer's material and are not redistributed in this repository.
