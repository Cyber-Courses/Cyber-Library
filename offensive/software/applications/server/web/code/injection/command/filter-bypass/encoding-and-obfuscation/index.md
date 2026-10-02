---
title: "Encoding and obfuscation bypass"
description: "Hiding the payload through runtime hex/ANSI-C decoding and case variation against case-sensitive blocklists."
keywords:
  - hex encoding
  - ANSI-C quoting
  - random case
  - obfuscation
  - WAF bypass
---

# Encoding and obfuscation

Hide the payload from naive parsers and case-sensitive blocklists: decode hex or ANSI-C sequences at runtime so the keyword never appears in the request, or vary casing where the target shell is case-insensitive, so the inspected bytes do not match the command that executes.

## Tools

- **[commix](https://github.com/commixproject/commix)**: tamper modules automate encoding and case obfuscation.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater and Decoder for crafting encoded payloads.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
