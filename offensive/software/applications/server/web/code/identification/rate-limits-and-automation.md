---
title: "Rate limits and automation: throttle bypass, credential stuffing, and password spraying"
description: "How weak or mis-keyed rate limiting opens credential endpoints to automation: per-IP vs per-account keying, header rotation, GraphQL/JSON batching, password spraying, and the single-packet race."
keywords:
  - rate limiting bypass
  - credential stuffing
  - password spraying
  - brute force
  - GraphQL batching
  - X-Forwarded-For
---

# Rate limits and automation

Rate limiting is the control that turns a theoretical brute force into an impractical one. When it is missing, mis-keyed, or inconsistently applied, credential endpoints become automatable, and the account list from [enumeration](account-enumeration.md) becomes account takeover. This page is about defeating the throttle and the attack shapes that avoid tripping it.

## What the limiter keys on

Every bypass starts with identifying the counter's key, then arranging for each request to carry a different value for it.

### Per-IP limits

If the limit counts requests per source IP, rotate the apparent source:

- **Forwarded headers**: many apps and middlewares trust `X-Forwarded-For`, `X-Real-IP`, `X-Client-IP`, `X-Originating-IP`, `Forwarded`, or `True-Client-IP`. Send a different (even random) value per request:

```
POST /login HTTP/1.1
X-Forwarded-For: 1.2.3.4
```

Rotate the value with Burp Intruder's pitchfork mode or a script. Some stacks honor a *duplicated* header, or take the first versus last value differently from the limiter, which is its own bypass.
- **Real rotation**: a proxy pool or a large IPv6 range gives genuinely distinct sources when headers are not trusted.

### Per-account limits

If the limit counts failures per account, do not hammer one account. **Password spraying** inverts the loop: try one plausible password against every account, then the next password, so no single account reaches its lockout threshold.

```
for each password in [Winter2026!, Company@123, Password1!]:
    for each user in confirmed_accounts:
        try(user, password)
    wait(lockout_window)
```

### Missing or inconsistent limits

The web login is often throttled while equivalent paths are not. Test the **API** (`/api/login`, `/oauth/token`), the **mobile** backend, legacy endpoints, and any alternate verifier. A limit on `/login` means nothing if `/api/v1/authenticate` has none.

## Batching many attempts into one request

Some backends count *requests*, not *attempts*, so packing many attempts into a single request slips past the counter:

- **GraphQL alias batching**: a single query can call the login mutation many times under different aliases, each with a different password, in one HTTP request:

```graphql
mutation {
  a: login(user:"victim", password:"p1"){ token }
  b: login(user:"victim", password:"p2"){ token }
  c: login(user:"victim", password:"p3"){ token }
}
```

- **JSON array / multi-value parameters**: endpoints that accept an array of credentials, or that iterate a repeated parameter, let one request try many values.

## Racing the counter

Where a limit exists but is incremented after the check, fire many requests in parallel so they are all validated before the counter catches up. Burp's **Turbo Intruder single-packet attack** (and HTTP/2 multiplexing) lands many requests in the same window, defeating a naive "N attempts then block" that is not atomic. This is the same primitive used against OTP verify endpoints.

## Exploitation

A credential-stuffing run against a confirmed account list, rotating the source header, with Turbo Intruder or a script:

```bash
ffuf -w creds.txt:CRED -u https://target/api/login -X POST \
  -H 'Content-Type: application/json' \
  -H 'X-Forwarded-For: FUZZRND' \
  -d '{"user":"CREDUSER","pass":"CREDPASS"}' \
  -mc 200
```

Combine with the enumeration output so you only try live accounts, which keeps volume (and lockout risk) down and success rate up.

## Tools

- **Burp Suite Intruder / Turbo Intruder**: payload sets, header rotation, single-packet race.
- **ffuf** / **hydra** / **medusa**: scripted credential attacks.
- Credential-stuffing word/combo lists built from the enumerated account set and breach corpora.

## References

- PortSwigger Web Security Academy: [Rate limiting bypass](https://portswigger.net/web-security/authentication) and [Bypassing rate limits via race conditions](https://portswigger.net/web-security/race-conditions)
- OWASP WSTG: [Testing for Weak Lock Out Mechanism (WSTG-ATHN-03)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism)
- OWASP: [Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing)
