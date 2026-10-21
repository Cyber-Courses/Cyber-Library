---
title: "User enumeration: discovering valid SSH usernames"
description: "Username enumeration is implementation- and version-specific. CVE-2016-6210 affected OpenSSH before 7.3 under particular password-hashing conditions, while CVE-2018-15473 affected OpenSSH through 7.7 through inconsistent malformed-request handling."
keywords:
  - ssh user enumeration
  - auth methods
  - timing attack
  - openssh
  - username
---

# User enumeration

SSH does not provide a standard username-enumeration operation. Account disclosure depends on a documented implementation flaw or a configuration-specific observable difference. CVE-2016-6210 affected OpenSSH before 7.3 when particular SHA-256 or SHA-512 password hashes were used: oversized password input could create a measurable timing difference between valid and invalid accounts. CVE-2018-15473 affected OpenSSH through 7.7 because malformed authentication requests were not parsed consistently for invalid users in several authentication handlers. OpenSSH 7.8 added countermeasures against account-validation timing attacks.

```bash
# authentication-method output is configuration evidence, not proof of account existence
ssh -vvv -o BatchMode=yes -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no user@<target> true 2>&1 \
  | sed -n 's/^debug1: Authentications that can continue: //p'
# test a named enumeration CVE only after confirming affected build and prerequisites
```

## Exploitation notes

- Authentication methods can vary with `Match` rules and account policy, but different output does not by itself prove whether a username exists.
- A banner is insufficient to establish vulnerability because vendor packages may backport fixes. Verify the installed package or vendor advisory as well as the upstream-looking version.
- CVE-2016-6210 timing tests require repeated samples and their documented hashing/input conditions; CVE-2018-15473 requires its malformed-request technique rather than timing analysis.
- Seed candidates from other sources (SMB RID cycling, web app usernames, email/OSINT, service-account conventions) and validate here before [password brute force](../authentication/password-brute-force.md).
- Authentication-method and vulnerability probes begin authentication exchanges and may be logged even when they do not submit a valid credential.

## Tools

- [OpenSSH release notes](https://www.openssh.com/releasenotes.html)

## References

- [CVE-2016-6210](https://www.cve.org/CVERecord?id=CVE-2016-6210)
- [OpenSSH 7.8 account-enumeration countermeasures](https://www.openssh.com/releasenotes.html#7.8)
- [CVE-2018-15473](https://www.cve.org/CVERecord?id=CVE-2018-15473)
