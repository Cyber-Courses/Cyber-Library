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

## Pages

- **[Character reconstruction](character-reconstruction.md)**: Rebuilding forbidden characters like / from environment-variable substrings (${HOME:0:1}), tr translation, and printf, so a filter blocking specific characte...
- **[Brace expansion](with-brace-expansion.md)**: Using Bash brace expansion, {cat,/etc/passwd}, {ls,-la}, to split a command into comma-separated tokens that carry no spaces and no filtered keyword, so a li...
- **[Tilde expansion](with-tilde-expansion.md)**: Rebuilding filtered paths from the shell's tilde expansion, ~ for HOME, ~+ for PWD, ~- for OLDPWD, so directory prefixes are produced by the shell rather tha...
- **[Variable and parameter expansion](with-variable-expansion.md)**: Assembling commands from shell parameter expansion, ${PATH:0:1} for /, ${IFS} for whitespace, substring and pattern substitution, so the forbidden characters...
- **[Wildcards](with-wildcards.md)**: Reconstructing filtered binary names and paths with shell glob characters, /???/c?t /???/p?sswd, /bin/c*, so the blocked literal never appears in the request...

## Tools

- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate expansion-based bypasses.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting expansion and glob payloads.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
