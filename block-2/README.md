# Block 2 — Authentication and Authorisation

**Deliverable:** [`SSC-Block2-Lab-Report-EN.pdf`](SSC-Block2-Lab-Report-EN.pdf), the individual lab report covering Labs 3, 4 and 5 (preparation questions, every exercise, and the reflections of the three labs).

**Environment:** the course lab VM (`ssc-lab`), running isolated in VirtualBox. Lab 4 runs a local OpenID Provider ([dex](https://dexidp.io/)) at `localhost:5556` and a demo client at `localhost:3000`. Lab 5 runs two Docker containers, `jump` and `target`.

| Lab | Topic | Folder |
|---|---|---|
| Lab 3 | PAM and the local authentication stack | [`lab-03-pam-and-the-authentication-stack/`](lab-03-pam-and-the-authentication-stack/) |
| Lab 4 | OIDC and federated identity | [`lab-04-oidc-and-federated-identity/`](lab-04-oidc-and-federated-identity/) |
| Lab 5 | SSH certificates, authorisation and hardening | [`lab-05-ssh-certificates-and-hardening/`](lab-05-ssh-certificates-and-hardening/) |

## Lab 3 — PAM and the authentication stack

I followed a login from the process that runs it down to the module that checks the password:

| Part | Question | What I did |
|---|---|---|
| A. Credentials and NSS | Who is a process, and where does an account come from? | Read the real, effective, saved and filesystem UIDs; traced account lookups through `nsswitch.conf`; explained why `/etc/passwd` is public and `/etc/shadow` is not |
| B. The PAM stack | How does a stack decide? | Read `/etc/pam.d/sudo` and `common-auth`; compared `required`, `sufficient` and `[success=1 default=ignore]` |
| C. Break it, then repair it | What happens when a module is missing? | Broke the `sudo` stack with an unknown module and confirmed that PAM fails closed |
| D. Hardening | How do lockout and password quality work? | Configured `pam_faillock` (3 failures, 120 s lockout) and `pam_pwquality` (12 characters, a digit and an uppercase letter) |

**Key finding:** account lockout slows online guessing to about 90 attempts per hour, but it is a denial-of-service lever: an attacker who only knows a username can keep that account locked. Placed in `common-auth`, it would lock every service on the machine.

## Lab 4 — OIDC and federated identity

I ran the authorisation-code flow (with PKCE, `state` and `nonce`) between a browser, a demo client and dex, and then attacked it:

| Part | What I did |
|---|---|
| A–B. The flow | Sketched the flow, captured the front-channel requests in the Network tab and showed that the code-for-token exchange never reaches the browser |
| C. What the tokens prove | Decoded the ID token, explained the security role of `iss`, `aud`, `exp`, `sub` and `nonce`, and matched the `kid` against the JWKS |
| D. `redirect_uri` validation | Tampered with the `redirect_uri` and showed that dex's exact matching rejects it, unlike prefix, wildcard and substring rules |
| E. ID token validation | Forged a token: the verifying client rejected it, the naive one accepted it |

**Key finding:** the demo client follows the protocol correctly but validates nothing in the ID token (no signature, `iss`, `aud`, `exp` or `nonce` check), so a forged token logs in as any user. "We use OIDC" describes the protocol, not the security. Even with a correct verifier, the client's `state` is not bound to the browser, which allows a login-CSRF attack; the lab folder includes a sketch of the cookie-based fix.

## Lab 5 — SSH certificates, authorisation and hardening

I moved two containers from plain public keys to an SSH certificate authority:

| Part | What I did |
|---|---|
| A. Plain keys | Installed keys by hand and showed the two operational problems: distribution and host authenticity (TOFU) |
| B. SSH CA | Issued user certificates, separated authentication (`TrustedUserCAKeys`) from authorisation (`AuthorizedPrincipalsFile`) and revoked one certificate with a KRL |
| C. Hardening `sshd` | Appended a five-line baseline, checked it with `sshd -T`, found the inert lines and fixed them |
| D. Agent forwarding | Hijacked a forwarded agent from `jump` as root, then mitigated it with `ProxyJump` |

**Key findings:**

- **Hardening that looks applied but is not.** `sshd` uses the first value it reads for each keyword, so four of the five lines I appended were ignored. The password test still failed, because of the one line that did apply, which hid the problem. Only `sshd -T` showed the effective configuration.
- **Agent forwarding hands over the key's use, not the key.** With `ssh -A`, any root on the bastion can sign as the user through the forwarded socket. `ssh -J` gives the same hop without exposing the agent.

## Report contents

- Preparation questions of the three labs
- **Lab 3:** Exercises 1a–7a
- **Lab 4:** Exercises 1a–5c, including the `client.py` audit and the claims table
- **Lab 5:** Exercises 1a–8a, with the commands I ran, the `ssh-keygen -L` output of both certificates and the final `sshd_config`
- **Appendix:** the reflections of Labs 3, 4 and 5

The keys, certificates and tokens shown in the screenshots were generated inside the isolated lab for this exercise and are not used anywhere else. The tokens have long expired.
