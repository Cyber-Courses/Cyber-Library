---
title: "Identification: account discovery, enumeration, and identifier exposure in web applications"
description: "How application behavior reveals which accounts exist, eases credential stuffing, and exposes user identifiers, the pre-authentication recon that feeds account takeover and access-control attacks."
keywords:
  - account enumeration
  - user enumeration
  - credential stuffing
  - identifier exposure
  - IDOR recon
---

# Identification

**Identification** is the recon layer beneath authentication: before you try to *become* a user, you work out *which* users exist, *which* credentials are worth trying, and *which* identifiers the application hands you for free. None of this defeats a credential on its own, but it is what turns a blind guess into a targeted attack. A confirmed username list plus a weak rate limit is a credential-stuffing run; a leaked sequence of user IDs is the input to every access-control test that follows.

This area is about the *signals the application emits*, not the authorization checks it enforces. It sits between [Authentication](../authentication/index.md) (proving identity) and Access Control (what an identity may do), and it feeds both.

## Where it breaks

Three recurring weaknesses give an attacker the recon they need:

- **Oracles that distinguish real from fake.** A login that says "unknown user" for one identifier and "wrong password" for another is an enumeration oracle. The same oracle hides in registration, password reset, MFA prompts, and even response timing.
- **Throttling that does not actually throttle.** Per-IP-only limits fall to header rotation and proxies; per-account limits fall to password spraying; API, GraphQL, and mobile endpoints are often exempt from the limits the web login enforces.
- **Identifiers handed out freely.** Sequential IDs in URLs, internal fields in API responses and exports, and public profile or autocomplete endpoints leak the very values an attacker needs to enumerate accounts and target objects.

## Attack methodology

1. **Collect candidate identifiers.** Harvest usernames and emails from public profiles, autocomplete and search endpoints, exports, API responses, and breach corpora.
2. **Find an existence oracle.** Compare responses for a known-valid and a known-invalid identifier across login, registration, reset, and SSO, holding the request shape identical, and measure timing when the bodies match.
3. **Confirm the account list.** Automate the oracle against your candidate list to produce a set of confirmed accounts.
4. **Measure the throttle.** Determine what the rate limit keys on (IP, account, session, nothing) and whether API/GraphQL/mobile paths share it.
5. **Feed the next stage.** Hand confirmed accounts to credential stuffing and password spraying, and hand leaked identifiers to access-control testing.

Enumeration and stuffing touch real accounts, so stay in scope and avoid locking out genuine users.

## Pages

- **[Account enumeration](account-enumeration.md)**: existence oracles in error messages, status codes, response shape, redirects, and timing, across login, registration, reset, and SSO.
- **[Rate limits and automation](rate-limits-and-automation.md)**: throttling gaps, per-IP vs per-account keying, GraphQL/JSON batching, password spraying, and the tooling that drives them.
- **[Identifier exposure in URLs and exports](identifier-exposure-in-urls-and-exports.md)**: sequential and predictable IDs, over-sharing APIs and exports, and public endpoints that leak user identifiers.
