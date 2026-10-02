---
title: "Expansion and globbing bypass"
description: "Rebuilding commands and paths through brace expansion, wildcard globbing, tilde expansion, and variable expansion."
keywords:
  - brace expansion
  - wildcard
  - globbing
  - tilde expansion
  - variable expansion
---

# Expansion and globbing

Reconstruct commands and paths through shell expansion rather than literal text: brace expansion, wildcard globbing (`/???/c?t /???/p?sswd`), tilde expansion, and variable or parameter expansion assemble the target from characters the blocklist is not watching for.

## Tools

- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate expansion-based bypasses.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting expansion and glob payloads.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
