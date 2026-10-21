---
title: "Username enumeration: discovering valid rlogin accounts"
order: 3
description: "Some rlogind and login/PAM combinations may reveal account validity through differing prompts, errors, or timing. This is implementation-dependent and must be established against the specific target before responses can be used for username enumeration."
keywords:
  - username enumeration
  - rlogin
  - login prompt
  - valid user
  - trust
---

# Username enumeration

`rlogin` may leak which usernames exist when a particular `rlogind`, login program, or PAM stack produces observably different prompts, errors, delays, or connection behavior for valid and invalid accounts. RFC 1282 does not define such an enumeration signal, and many login implementations deliberately normalize authentication responses. First compare known-valid and known-invalid control accounts under repeatable conditions; only treat a difference as enumeration after ruling out trust rules, rate limiting, network variance, and generic login failures. A passwordless shell demonstrates both a valid account and satisfied trust, but a password prompt alone does not universally prove that the supplied username exists.

```bash
# compare candidates only after establishing known-valid and known-invalid baselines
for u in <known-valid> <known-invalid> <candidate>; do
  echo "== $u =="; rlogin -l "$u" <target> </dev/null 2>&1 | head -2; done
# repeat probes and compare prompts, errors, timing, and connection handling;
# a distinction is useful only when the target implementation exposes one consistently
```

## Exploitation notes

- A user who reaches a passwordless shell is valid and trusted. Other prompts and failures are meaningful only after the target implementation has been shown to distinguish valid from invalid accounts.
- Seed candidates from legacy-system conventions (`root`, `oracle`, `operator`, `bin`, application accounts) and any other enumeration, then validate here.
- A discovered already-trusted account is a direct passwordless login, so username enumeration can surface the easiest path, not just a target list.
- Feed only reliably validated users into [rhosts bypass](rhosts-bypass.md) assessment or authorized credential testing.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
