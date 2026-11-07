# Lab 5 — SSH Certificates, Authorisation and Hardening

**Goal:** move two containers, `jump` and `target`, from plain SSH public keys to an SSH certificate authority, separate authentication from authorisation, revoke a certificate, harden `sshd` and see how agent forwarding can be hijacked.

The full answers are in the [Block 2 report](../SSC-Block2-Lab-Report-EN.pdf), Section 3.

## What I did

| Part | Exercises | Summary |
|---|---|---|
| A. Key-based authentication and its limits | 1a | Installed the `dev` key by hand on both containers and accepted two fingerprints blindly (TOFU) |
| B. An SSH certificate authority | 2a–5b | Created a user CA and issued short-lived certificates (`dev`, serial 1; `ops`, serial 2); trusted the CA with `TrustedUserCAKeys`; authorised principals per account with `AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u`; revoked the `ops` certificate with a KRL (`RevokedKeys`) |
| C. Hardening `sshd` | 6a–6c | Appended a five-line baseline, compared it with `sshd -T`, found four inert lines, fixed them and ranked the directives |
| D. The agent-forwarding hijack | 7a, 8a | As root on `jump`, used the victim's forwarded agent socket to log in to `target` as `dev`; mitigated with `ProxyJump` (`ssh -J`) |

## Key commands

```bash
# Create the user CA (pre-lab)
ssh-keygen -t ed25519 -f ~/.ssh/user_ca -N '' -C 'SSC user CA'

# Sign the ops key: Key ID student-ops, principal ops, serial 2, valid for one hour
ssh-keygen -s ~/.ssh/user_ca -I 'student-ops' -n ops -z 2 -V +1h ~/.ssh/id_ops.pub

# Inspect a certificate
ssh-keygen -Lf ~/.ssh/id_ops-cert.pub

# Show the effective sshd configuration (the only reliable check)
docker compose exec target sshd -T

# Reach target through jump without forwarding the agent
ssh -J jump dev@target
```

Installing the `dev` public key by hand, from the VM, as root of the container (the same for `jump`):

```bash
docker compose exec -T target sh -c 'mkdir -p /home/dev/.ssh \
&& cat > /home/dev/.ssh/authorized_keys \
&& chown -R dev:dev /home/dev/.ssh \
&& chmod 700 /home/dev/.ssh && chmod 600 /home/dev/.ssh/authorized_keys' < ~/.ssh/id_dev.pub
```

## Findings

- **Four mechanisms, four different jobs.** The CA signature sets identity, `AuthorizedPrincipalsFile` sets which principals each account accepts, the validity interval limits exposure in time, and only the KRL ends a certificate before it expires.
- **Hardening that looks applied but is not.** `sshd` keeps the first value it reads for each keyword. A block of explicit defaults earlier in the file made four of my five appended lines inert. The password test still failed because of the one line that did apply (`AuthenticationMethods publickey`), which hid the problem. Only `sshd -T` showed it.
- **Agent forwarding hands over the use of the key, not the key.** With `ssh -A`, any root on the bastion can sign through the forwarded socket. `ssh -J` gives the same hop without exposing the agent. When `-A` is unavoidable, load the key with `ssh-add -c` and set `ForwardAgent no` per host.

## Course material

- Handout: [`lab-05-ssh-certificates-and-hardening.pdf`](lab-05-ssh-certificates-and-hardening.pdf)
- Lecture: [`lecture-05-authorisation-and-ssh.pdf`](../lectures/lecture-05-authorisation-and-ssh.pdf)

Both were written by the course lecturer, who holds the rights to them.
