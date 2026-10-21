---
title: "IPMI RAKP hash disclosure: pre-auth password-hash retrieval"
order: 2
description: "The IPMI 2.0 RAKP authentication exchange can return a password-keyed HMAC for a valid account before authentication completes. Exposed implementations may therefore let an unauthenticated attacker test likely usernames, capture RAKP material, and attempt offline password recovery."
keywords:
  - rakp
  - ipmi 2.0
  - hash disclosure
  - offline cracking
  - bmc
---

# IPMI RAKP hash disclosure

The weakness follows from the IPMI 2.0 RAKP (RMCP+ Authenticated Key-Exchange Protocol) design. During authentication, a BMC can return a password-keyed HMAC in RAKP message 2 before the client has authenticated. Against an exposed implementation, an attacker requests an exchange for candidate account names and captures the material for valid accounts, then tests password candidates offline. The result depends on the BMC exposing RMCP+, accepting the requested authentication exchange, and the supplied username identifying a valid account; it should not be described as guaranteed leakage for every account on every IPMI 2.0 device.

```bash
# retrieve the RAKP hash for a user and crack it offline
#   Metasploit module automates the RAKP exchange and outputs a hash
#   auxiliary/scanner/ipmi/ipmi_dumphashes  (RHOSTS, USER_FILE)
# the output is a crackable format:
#   <user>:<16-byte salt/exchange data>:<HMAC>  -> feed to hashcat mode 7300 (IPMI2 RAKP)
hashcat -m 7300 ipmi_rakp.hashes wordlist.txt
```

## Exploitation notes

- This is a pre-authentication, protocol-design weakness, but retrieval still depends on the implementation, exposed IPMI channel, and a valid account name. Test likely vendor defaults and an authorized username list rather than assuming `root` or `ADMIN` exists.
- The retrieved RAKP exchange material contains a password-keyed HMAC suitable for offline guessing with hashcat mode 7300; success then depends on password strength and the candidate set.
- A cracked BMC password gives authenticated administrative control of the controller (power, console, virtual media), the same impact as the cipher 0 bypass but via a recovered credential.
- Enumerate usernames first (defaults and any disclosed); combine with [BMC default credentials](bmc-default-credentials.md) (the cracked password is often a default anyway) and [cipher 0](ipmi-cipher-0-bypass.md).

## Tools

- [Metasploit ipmi_dumphashes](https://www.metasploit.com/)
- [hashcat (mode 7300, IPMI2 RAKP)](https://hashcat.net/hashcat/)

## References

- [Dan Farmer: IPMI RAKP hash disclosure](http://fish2.com/ipmi/remote-pw-cracking.html)
- [IPMI 2.0 RAKP](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
