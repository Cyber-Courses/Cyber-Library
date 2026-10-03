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

## Pages

- **[Backslash + newline continuation](with-backslash-newline.md)**: Using backslash followed by a newline to break a keyword or command across physical lines so a literal blocklist misses it while the shell rejoins and execut...
- **[Newline as a command terminator](with-line-return.md)**: Injecting a raw or URL-encoded newline (%0a, \\n) to terminate the intended command and start a new one inside sh -c, bypassing filters that only block ; | &.
- **[Spaceless payloads](without-space.md)**: Producing whoami/id when the space character is filtered, using ${IFS}, $IFS$9, brace expansion, input redirection, and tab substitutes to separate a command...

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting space-free payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate no-space bypasses.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
