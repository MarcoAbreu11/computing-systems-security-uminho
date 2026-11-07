# Lab 4 — OIDC and Federated Identity

**Goal:** run the OpenID Connect authorisation-code flow between a browser, a demo client (`localhost:3000`) and a local OpenID Provider, [dex](https://dexidp.io/) (`localhost:5556`), then break it through two common misconfigurations.

The full answers are in the [Block 2 report](../SSC-Block2-Lab-Report-EN.pdf), Section 2.

## What I did

| Part | Exercises | Summary |
|---|---|---|
| A. The actors and the flow | 1a | Sketched the flow and marked which messages the browser sees (front channel) and which it does not (the code-for-token exchange on the back channel) |
| B. Run the flow and capture the tokens | 2a–2c | Recorded the authorisation request (`state`, `nonce`, PKCE `S256`), captured the tokens and audited `client.py` against the checks a client must make |
| C. What the tokens prove | 3a–3c | Explained the security role of `iss`, `aud`, `exp`, `sub` and `nonce`; matched the token's `kid` against dex's three-key JWKS; contrasted ID and access tokens |
| D. `redirect_uri` validation | 4a–4d | Sent a tampered `redirect_uri` (`http://localhost:3000.attacker.example/callback`): dex's exact match rejected it with `400 Unregistered redirect_uri`. Explained the code-theft attack under prefix, wildcard and substring rules, and what PKCE does and does not stop |
| E. ID token validation | 5a–5c | Forged a token (`sub` and `email` changed): `verify.py` rejected it, `verify_naive.py` accepted it. Listed the minimum validations in order: algorithm, signature, claims |

## Findings

- **The demo client checks nothing in the ID token:** no signature, `iss`, `aud`, `exp` or `nonce` check, and it identifies users by email. A forged token logs in as anyone. "We use OIDC" describes the protocol, not the security.
- **Exact `redirect_uri` matching is the last line of defence.** With a public client, the attacker generates the PKCE verifier; only exact matching keeps the code away from the attacker's address.
- **`alg: none` and "just decode the JWT" are the same flaw:** the signature is never checked against a trusted key.
- **A valid token does not prove who is at the browser.** Because `state` is not bound to the browser, a login-CSRF attack still works with a correct verifier. [`client-state-fix.md`](client-state-fix.md) sketches the cookie-based fix.

## Course material

Handout: `lab-04-oidc-and-federated-identity.pdf`. It is the lecturer's material and is not redistributed in this repository.
