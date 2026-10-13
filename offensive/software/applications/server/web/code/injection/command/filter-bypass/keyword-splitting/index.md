---
title: "Keyword splitting bypass"
order: 2
description: "Breaking a blocked keyword into pieces the shell rejoins before execution, quotes, backslashes, empty substitutions, positional parameters."
keywords:
  - keyword splitting
  - quote splitting
  - blocklist bypass
  - obfuscation
  - command injection
---

# Keyword splitting

Break a blocked keyword into fragments the shell removes before execution. Quotes (`w"h"o"a"m"i`), backslashes, empty backticks or `$()`, and positional-parameter tricks all insert characters the shell deletes during expansion, so a literal-string blocklist never sees the forbidden word.

## Pages

- **[Backslash and slash insertion](with-backslash-and-slash.md)**: Breaking a blocked keyword with backslash escapes, w\\ho\\am\\i, and inserting redundant slashes into paths, /bin/c\\at, //bin//cat, so the shell normalizes...
- **[Empty backticks](with-backticks.md)**: Splitting a filtered command keyword with an empty backtick substitution, wh``oami, so the shell removes the empty command and reassembles the keyword, defea...
- **[Positional parameters ($@, $0)](with-dollar-at.md)**: Breaking a blocked keyword by inserting positional-parameter expansions that evaluate to nothing, who$@ami, who${@}ami, and abusing $0 as a shell invocation,...
- **[Empty command substitution ($())](with-dollar-parentheses.md)**: Breaking a blocked keyword by inserting an empty $() command substitution, who$()ami, so the shell runs nothing, removes it, and reassembles the keyword, byp...
- **[Double quotes](with-double-quote.md)**: Breaking a blocked command keyword with empty double-quote pairs, w\"h\"o\"a\"m\"i, so the shell strips the quotes and reassembles the word, bypassing a lite...
- **[Single quotes](with-single-quote.md)**: Breaking a blocked command keyword with empty single-quote pairs, w'h'o'a'm'i, so the shell strips the quotes and reassembles the word, bypassing a literal b...

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting split-keyword payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate keyword-splitting bypasses.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
