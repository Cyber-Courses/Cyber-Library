---
title: "SNMPv3 attacks: username enumeration, cracking, and downgrade"
description: "SNMPv3 adds user authentication and privacy, but it still leaks: the engine-discovery and report exchange confirms whether a username is valid before authentication, enabling user enumeration, and a captured authenticated exchange is cracked offline for weak auth passwords. Where v1/v2c remain enabled, the simplest attack is to downgrade to them entirely."
keywords:
  - snmpv3
  - username enumeration
  - usm
  - offline cracking
  - downgrade
---

# SNMPv3 attacks

SNMPv3 replaces the community string with user-based security (USM): a username, an authentication key (MD5 or SHA), and optionally a privacy key for encryption. This removes the cleartext-string weakness, but SNMPv3 is still attackable. First, username enumeration: the engine-discovery handshake and the report PDUs the agent returns distinguish a valid username from an invalid one before authentication completes, so an attacker confirms real usernames by probing. Second, offline cracking: a captured authenticated request contains the HMAC over known data, so weak authentication passwords are recovered offline. Third, and simplest, downgrade: if the device also still answers v1/v2c (common), ignore v3 and attack the cleartext path.

```bash
# username enumeration: valid vs invalid users elicit different engine/report responses
snmpget -v3 -l authNoPriv -u <guess> -a SHA -A badpass <target> 1.3.6.1.2.1.1.1.0 2>&1
#   "Unknown user name" vs "Authentication failure" distinguishes invalid vs valid user
# tools automate the enumeration (e.g. Metasploit snmp_login / snmpv3 enum modules)
# offline cracking of captured v3 auth (weak MD5/SHA auth password)
#   capture an authenticated request, extract the USM fields, crack with a wordlist
# downgrade: if v1/v2c also answer, attack them instead (see version detection)
```

## Exploitation notes

- The agent's different responses to an unknown username versus a known username with a bad password are the enumeration oracle; a validated username is the target for offline cracking or for guessing the auth password.
- Offline cracking works against weak authentication passphrases (the HMAC is over known plaintext); strong auth/priv keys resist it, so success depends on password strength.
- Downgrade is usually the path of least resistance: enterprises leave v1/v2c enabled for legacy tools, so confirm the versions ([Version detection](version-detection.md)) and prefer the cleartext route when available.
- A recovered v3 user with auth (and priv) grants full read and, depending on the user's view, write; treat it as you would any device credential.

## References

- [RFC 3414 (SNMPv3 USM)](https://datatracker.ietf.org/doc/html/rfc3414)
- [HackTricks: SNMPv3](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
