---
title: "Attacking OTP and MFA: brute-forcing codes, rate-limit bypass, and 2FA logic flaws"
description: How a pentester defeats second-factor authentication—brute-forcing weak codes, bypassing rate limits, skipping the MFA step, and abusing codes not bound to the user—with example requests, detection, and remediation.
keywords:
  - MFA bypass
  - OTP brute force
  - 2FA
  - rate limiting
  - one-time password
  - authentication
---

# Weak OTP and MFA throttling

Application-level weaknesses in second-factor authentication that let an attacker who has (or fakes) the first factor reach a fully authenticated session. The flaws fall into three families: **brute-forcing** weak one-time codes, **bypassing** the rate limits meant to stop that, and **logic flaws** where the MFA step can be skipped or the code is not bound to the right user.

> **Scope.** Authorized testing only. MFA testing targets real account-protection controls—stay within engagement scope and avoid locking out genuine users.

## Overview

The canonical flow is two-stage: stage one validates `username:password`; stage two validates a second factor (a TOTP from an authenticator app, an SMS/email OTP, a push approval, or a backup code). The security-critical invariant is that **stage two must be statefully bound to the exact user who passed stage one, and the post-MFA session must be unusable until stage two succeeds.**

PortSwigger frames the core design flaw: when the user enters a password and is then prompted for a code on a separate page, they are effectively "logged in" before the code is checked. A server that issues a session identifying the pending user *before* verifying the second factor is the root cause of most breaks below. The keyspace compounds it—a 4-digit code has 10,000 values, a 6-digit code ~1,000,000—so without server-side throttling the second factor degrades to a brute-force-solvable challenge.

## Exploitation

### Brute-forcing the OTP

With a short numeric code and no enforced attempt cap, enumerate the keyspace against the verify endpoint. OWASP warns that apps often accept codes from a window on either side of the current TOTP (previous/current/next), multiplying your odds; if several codes are simultaneously valid, sustained guessing reaches high success probability within hours.

A representative verify request to drive with Burp Intruder:

```
POST /login2 HTTP/1.1
Cookie: session=<post-password session>
Content-Type: application/x-www-form-urlencoded

mfa-code=0000
```

Set the payload position on `mfa-code`, generate `0000–9999`, and use a session-handling **macro** to replay the preceding login requests so each guess carries a fresh valid session (apps that log you out after two wrong codes are defeated this way—the protection is cosmetic). Success is typically an HTTP **302** to the account page.

### Bypassing the rate limit

When a limiter exists, attack its key rather than the code:

- **Rotate the client identity** if the limiter is IP-keyed—spoof `X-Forwarded-For` (also `X-Real-IP`, `X-Client-IP`, `Forwarded`) per request; test non-standard headers (`X-Debug`) that may disable MFA entirely.
- **Reset the counter**—re-request a fresh code, or change the `username`/account parameter so the counter keys on a different identity.
- **Tamper the throttle parameter**—where a request field carries the limit state, send it empty or null.
- **Concurrency / race**—fire many verify requests in parallel (Turbo Intruder single-packet attack) so they are checked before the counter increments.

### Skipping the MFA step (logic flaws)

Because the session may already be "logged in," force-browse straight to a post-MFA endpoint (`GET /my-account`) and see whether it loads without the second step. Tamper response-driven client logic (`"verified":false`→`true`), watch for status-code tells (302 vs 200), or route into an MFA-less flow (OWASP cites changing an Azure AD B2C policy `B2C_1_SignInWithMFA`→`B2C_1_SignIn`).

### Code not bound to the user

PortSwigger's flagship example: stage one sets `Set-Cookie: account=carlos`, and the verify request trusts that cookie (or a `verify` body parameter) to decide *whose* code is checked. Change it to a victim and you generate and validate a code against an account you never authenticated as:

```
POST /login-steps/second HTTP/1.1
Cookie: account=victim-user

verify=victim-user&mfa-code=1234
```

Related logic flaws: code reuse (no single-use), no expiry (stale codes accepted), and codes valid across users or sessions.

### Tokens around MFA

- **Leaked OTP**—inspect the issuance response body/headers; codes are sometimes returned for debugging.
- **Backup / recovery codes**—often longer-lived and weakly throttled; brute-force or replay them and check single-use.
- **"Remember device" / stay-logged-in cookies**—if forgeable, they bypass MFA. PortSwigger's lab cookie is `base64(username + ':' + md5(password))`, an unsalted MD5 of the password that can be brute-forced *offline*.
- **SMS / SIM-swap** (note briefly)—SMS OTP is vulnerable to SS7 interception and SIM-swap; also test changing a phone-number parameter so the code is delivered to an attacker-controlled number.

## Detection (code review)

Verify server-side, not client-side: (1) is the attempt counter enforced server-side and reset only on success—never by re-requesting a code or changing identity headers? (2) is the code bound to the pending user/session in server state, not a client-supplied `account`/`verify`/`username`? (3) short expiry and single-use? (4) is the limiter keyed on the trusted session/account rather than a spoofable `X-Forwarded-For`? Grep `mfa`, `otp`, `2fa`, `verify`, `validateOtp`, `==`/`equals` on codes, `X-Forwarded-For`, `remember`, `backup_code`, `attempts`, `rate_limit`, and any post-password endpoint lacking an MFA-state guard.

## Remediation

Enforce **server-side attempt limits** with lockout or exponential backoff, keyed per-account *and* per-IP, invalidating the pending session on lockout. **Bind the code to session + user** server-side—never trust a client parameter to select the account. Use **short expiry** and **single-use** codes, accept only the current TOTP step, and prefer app-generated TOTP over SMS. Use **constant-time** comparison, never expose codes in responses or logs, and **enforce the MFA state machine server-side** so protected resources reject "password-passed, MFA-pending" sessions. Secure backup codes (single-use, throttled) and make "remember device" tokens unforgeable. Do not honor untrusted `X-Forwarded-For` for rate limiting.

## Tools

- **Burp Suite** — Intruder (code brute force), Turbo Intruder (race/concurrency), session-handling macros, and the **Param Miner** extension for header discovery.

## References

- PortSwigger — [Vulnerabilities in multi-factor authentication](https://portswigger.net/web-security/authentication/multi-factor) and [other authentication mechanisms](https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie)
- OWASP WSTG — [Testing Multi-Factor Authentication (WSTG-ATHN-11)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/11-Testing_Multi-Factor_Authentication) and [Weak Lock Out Mechanism (WSTG-ATHN-03)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism)
- OWASP — [Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)

## See also

- [Authentication (parent)](index.md)
- [Rate limit and automation](../identification/rate-limit-and-automation.md) — the throttling weaknesses these attacks exploit.
- [Race conditions in parallel effects](../workflow/parallel-effects/index.md) — concurrency bypass of attempt counters.
