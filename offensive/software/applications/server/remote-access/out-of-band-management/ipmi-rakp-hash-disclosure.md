---
title: "IPMI RAKP hash disclosure: pre-auth password-hash retrieval"
description: "The IPMI 2.0 RAKP authentication exchange returns a salted HMAC of the requested user's password before authentication completes, by design. An attacker requests the exchange for a username and receives a crackable hash, so any BMC speaking IPMI 2.0 leaks a password hash for every account to an unauthenticated attacker for offline cracking."
keywords:
  - rakp
  - ipmi 2.0
  - hash disclosure
  - offline cracking
  - bmc
---

# IPMI RAKP hash disclosure

The flaw here is in the IPMI 2.0 specification itself, not a given implementation. The RAKP (RMCP+ Authenticated Key-Exchange Protocol) handshake has the BMC return, in RAKP message 2, a salted HMAC computed over the requested user's password, before the client has proven anything. So an attacker who initiates the exchange for a username receives a hash of that account's password and cracks it offline. Because it is specification-mandated, essentially every BMC speaking IPMI 2.0 is affected: an unauthenticated attacker retrieves a crackable password hash for root and any other account, and weak BMC passwords (common) fall quickly.

```bash
# retrieve the RAKP hash for a user and crack it offline
#   Metasploit module automates the RAKP exchange and outputs a hash
#   auxiliary/scanner/ipmi/ipmi_dumphashes  (RHOSTS, USER_FILE)
# the output is a crackable format:
#   <user>:<16-byte salt/exchange data>:<HMAC>  -> feed to hashcat mode 7300 (IPMI2 RAKP)
hashcat -m 7300 ipmi_rakp.hashes wordlist.txt
```

## Exploitation notes

- This is pre-authentication and specification-level, so it applies to almost all IPMI 2.0 BMCs; the attacker needs only to know or guess usernames (`root`, `ADMIN`, `admin` are near-universal) to request their hashes.
- The retrieved value is a salted HMAC crackable offline (hashcat mode 7300); BMC passwords are frequently defaults or weak, so cracking commonly succeeds.
- A cracked BMC password gives authenticated administrative control of the controller (power, console, virtual media), the same impact as the cipher 0 bypass but via a recovered credential.
- Enumerate usernames first (defaults and any disclosed); combine with [BMC default credentials](bmc-default-credentials.md) (the cracked password is often a default anyway) and [cipher 0](ipmi-cipher-0-bypass.md).

## Tools

- [Metasploit ipmi_dumphashes](https://www.metasploit.com/)
- [hashcat (mode 7300, IPMI2 RAKP)](https://hashcat.net/hashcat/)

## References

- [Dan Farmer: IPMI RAKP hash disclosure](http://fish2.com/ipmi/remote-pw-cracking.html)
- [IPMI 2.0 RAKP](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
