---
title: "Account enumeration: existence oracles in errors, status, response shape, and timing"
description: "How to confirm which accounts exist by comparing application responses across login, registration, password reset, and SSO, including message, status, body, redirect, and timing oracles."
keywords:
  - account enumeration
  - user enumeration
  - username enumeration
  - timing attack
  - password reset enumeration
---

# Account enumeration

Account enumeration is confirming whether a given identifier (username, email, phone) corresponds to a real account, before and without authenticating. The application becomes an **existence oracle** whenever its behavior differs between a valid and an invalid identifier. The output is a list of confirmed accounts that feeds credential stuffing, password spraying, and targeted phishing.

## Oracles to test

The difference between "account exists" and "account does not" leaks through several channels. Test each by sending one request with a known-valid identifier and one with a known-invalid identifier, keeping everything else identical, and diffing the results.

### Error-message differences

The classic oracle is distinct text:

- **Login**: "Unknown username" versus "Incorrect password" tells you the username is valid on the second message. Even a generic "invalid credentials" can leak elsewhere.
- **Registration**: "That email is already registered" directly confirms existence. Registration is often the most reliable oracle because it must tell the user about collisions.
- **Password reset**: an app that says "No account with that email" for invalid input but "Check your inbox" for valid input is an oracle, even when it tries to be discreet.

### Status, body, and header differences

When messages are uniform, smaller tells remain:

- **HTTP status code**: 200 versus 302, or 200 versus 400.
- **Response length**: a few bytes of difference (a hidden field, a different token, a whitespace change) is enough; sort Burp Intruder results by length.
- **Field-level validation errors** in JSON APIs: `{"error":"user_not_found"}` versus `{"error":"invalid_password"}`.
- **Set-Cookie or redirect target**: a valid login may set a pending-MFA cookie or redirect to `/mfa`, while an invalid one returns to `/login`.

### Flow-branching oracles

Multi-step flows branch on existence:

- **MFA prompt**: if the second-factor page appears only for valid accounts (because invalid ones fail at stage one), the prompt itself is the oracle.
- **SSO / magic-link**: "we emailed you a link" for valid users versus an immediate error for unknown ones.
- **Forgot-username** and invite flows often confirm existence explicitly.

### Timing oracles

When responses are byte-identical, timing often is not. If the server runs an expensive password hash (bcrypt, argon2) only when the username exists, valid usernames are measurably slower; if it short-circuits on an unknown user, invalid ones are faster. Collect many samples per identifier to average out network jitter, then compare distributions rather than single requests.

## Exploitation

Build a candidate list, then drive the chosen oracle with an automation tool. A login-error oracle with Burp Intruder or ffuf:

```bash
ffuf -w users.txt -u https://target/login -X POST \
  -d 'username=FUZZ&password=Wrong123!' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -mr 'Incorrect password'      # match the "valid user, wrong password" branch
```

Matching on the valid-user message (`-mr`) returns exactly the confirmed accounts. Where the tell is length rather than text, filter on size instead (`-fs`/`-ms`). For reset or registration oracles, point the same technique at `/forgot-password` or `/register` and match the existence-confirming branch.

For a timing oracle, issue repeated requests per identifier and compare median response times:

```bash
for u in $(cat users.txt); do
  t=$(curl -s -o /dev/null -w '%{time_total}' -X POST https://target/login \
        -d "username=$u&password=Wrong123!")
  echo "$t  $u"
done | sort -rn | head
```

Enumeration is often possible across several endpoints at once; the most reliable oracle on a given target is frequently registration or reset rather than login, so test all of them before concluding the app is safe.

## Tools

- **Burp Suite Intruder**: drive the oracle, sort by status/length/time.
- **ffuf** / **wfuzz**: fast match-and-filter on message or size.
- **Username harvesting**: public profiles, `theHarvester`, search and autocomplete endpoints, breach corpora.

## References

- PortSwigger Web Security Academy: [Username enumeration](https://portswigger.net/web-security/authentication/password-based)
- OWASP WSTG: [Testing for Account Enumeration and Guessable User Account (WSTG-IDNT-04)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account)
