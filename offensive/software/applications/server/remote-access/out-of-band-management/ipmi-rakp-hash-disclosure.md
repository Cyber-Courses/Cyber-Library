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
# save the module's hashcat-compatible output, then use IPMI2 RAKP mode 7300
hashcat -m 7300 ipmi_rakp.hashes wordlist.txt
```

## Exploitation notes

- This is a pre-authentication, protocol-design weakness, but retrieval still depends on the implementation, exposed IPMI channel, and a valid account name. Test likely vendor defaults and an authorized username list rather than assuming `root` or `ADMIN` exists.
- The retrieved RAKP exchange material contains a password-keyed HMAC suitable for offline guessing with hashcat mode 7300; success then depends on password strength and the candidate set.
- A recovered password provides authenticated access bounded by that account's role and channel privilege. Confirm effective permissions rather than assuming administrative power, console, or virtual-media access.
- Test authorized candidate usernames and passwords; relate the result to [BMC default credentials](bmc-default-credentials.md) and [cipher 0](ipmi-cipher-0-bypass.md) without assuming the weaknesses coexist.

## Tools

- [Metasploit ipmi_dumphashes](https://www.metasploit.com/)
- [hashcat (mode 7300, IPMI2 RAKP)](https://hashcat.net/hashcat/)

## References

- [Dan Farmer: IPMI RAKP hash disclosure](http://fish2.com/ipmi/remote-pw-cracking.html)
- [IPMI 2.0 RAKP](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
