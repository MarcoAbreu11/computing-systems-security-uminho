# Fixing the `state` storage in `client.py`

In `client.py`, the `state` is stored in a global dictionary (`PENDING`), with no link to the browser that started the login. At `/callback`, the client only checks whether the `state` exists in `PENDING`, not whether it was issued to the browser presenting it. The fix binds the `state` to the browser's session through a cookie created at `/login`.

## Code change (sketch)

```python
# /login
pre = secrets.token_urlsafe(16)
PENDING[pre] = {"state": state, "verifier": verifier, "nonce": nonce}
self._send(302, "", [("Location", f"{DEX}/auth?{params}"),
                     ("Set-Cookie", f"pre={pre}; Path=/; HttpOnly")])

# /callback
pre = <cookie "pre" sent by the browser>
pend = PENDING.pop(pre, None)
if not pend or pend["state"] != state:
    <reject the request>
```

Instead of checking whether the `state` exists, the client checks whether it is the `state` it issued to this browser.

## Normal flow (the victim logs in)

1. The victim's browser requests `/login`.
2. The client generates `state-V`, replies with a redirect to dex and sends the cookie `pre=V1`. It stores `PENDING[V1] = state-V`.
3. The victim authenticates at dex, which redirects them to `/callback?code=…&state=state-V`.
4. The browser requests `/callback?code=…&state=state-V` and automatically sends the cookie `pre=V1`.
5. The client finds `state-V` in `PENDING[V1]`, equal to the one in the URL, and accepts.
6. The client exchanges the code for tokens, validates the ID token with `verify.py` and creates the session.

## The login-CSRF attack, with the fix

1. The attacker requests `/login` in their browser. The client generates `state-A`, sends the cookie `pre=A1` to the attacker's browser and stores `PENDING[A1] = state-A`.
2. The attacker authenticates at dex with their own account and copies the URL `/callback?code=…&state=state-A`, without opening it.
3. The attacker convinces the victim to open that URL.
4. The victim's browser requests `/callback?code=…&state=state-A`, but sends its own cookie (`pre=V1`) or none. It does not have `pre=A1`, which is in the attacker's browser.
5. The client compares `state-A` with the `state` associated with the victim's cookie. They do not match, and the client rejects the request.
6. The code is not exchanged and no session is created: the victim is not logged in to the attacker's account.

## Conclusion

The attacker can put their `state` in the URL the victim opens, but cannot put their cookie in the victim's browser. Requiring the `state` in the URL to match the browser's cookie ensures the login is completed by the same browser that started it. The same applies to the `nonce`: since it is stored in `PENDING[pre]`, the expected `nonce` also becomes the one for this browser.
