---
title: "Username enumeration: discovering valid rlogin accounts"
description: "rlogin's login flow can reveal which usernames are valid: the service's handling of trusted versus untrusted and existing versus non-existent accounts differs, so the prompt behaviour distinguishes real users. A validated user list focuses trust-abuse and password attacks on accounts that exist."
keywords:
  - username enumeration
  - rlogin
  - login prompt
  - valid user
  - trust
---

# Username enumeration

rlogin can leak which usernames exist through differences in its login handling. Depending on the implementation, a trusted-but-nonexistent user, an existing user without trust (prompted for a password), and a nonexistent user (rejected differently) produce distinguishable responses, so probing the `-l <user>` behaviour separates real accounts from invalid ones. A validated user list then focuses the trust-abuse and password attacks, you plant `.rhosts` for, or spray/capture against, accounts that actually exist, and reveals which accounts are already trusted (logging in with no password).

```bash
# probe usernames and classify the response
for u in root admin oracle bin operator; do
  echo "== $u =="; rlogin -l "$u" <target> </dev/null 2>&1 | head -2; done
# distinguish: immediate shell (trusted+exists), password prompt (exists, no trust),
#   rejection/error (invalid user) - the differences enumerate valid accounts
```

## Exploitation notes

- The useful distinction is three-way: a user who logs in with no password is both valid and trusted (immediate win), a user prompted for a password is valid but untrusted (target for capture/planting), and a differing rejection marks an invalid user.
- Seed candidates from legacy-system conventions (`root`, `oracle`, `operator`, `bin`, application accounts) and any other enumeration, then validate here.
- A discovered already-trusted account is a direct passwordless login, so username enumeration can surface the easiest path, not just a target list.
- Feed validated users into [rhosts bypass](rhosts-bypass.md) (plant trust) and [cleartext password](cleartext-passwords.md) capture.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
