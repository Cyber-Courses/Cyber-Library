---
title: "Spaceless and whitespace bypass"
description: "Defeating space- and line-based filters with ${IFS}, brace expansion, and newline or backslash-newline continuations."
keywords:
  - spaceless
  - IFS
  - no space
  - newline injection
  - line continuation
---

# Whitespace

Techniques that defeat filters targeting the space character or line structure: `${IFS}`, brace expansion, and input redirection separate arguments without a literal space. An unescaped newline terminates a command inside a spawned shell, while a backslash-newline continuation is removed and the two physical lines are joined before tokenization, useful for hiding a token across lines rather than terminating the command.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting space-free payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate no-space bypasses.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
