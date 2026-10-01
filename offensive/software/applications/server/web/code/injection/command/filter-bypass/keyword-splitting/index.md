---
title: "Keyword splitting bypass"
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
