---
title: "Keyword splitting with backslash and slash: w\\ho\\am\\i and /bin/c\\at"
description: "Breaking a blocked keyword with backslash escapes, w\\ho\\am\\i, and inserting redundant slashes into paths, /bin/c\\at, //bin//cat, so the shell normalizes the token and a literal blocklist misses it."
keywords:
  - command injection
  - keyword splitting
  - backslash
  - slash
  - filter bypass
  - path normalization
---

# Backslash and slash insertion

Two normalization quirks let an attacker rewrite a filtered keyword or path so it no longer matches a signature yet still resolves to the same command. A **backslash** escapes the following character, and for an ordinary letter the shell simply drops the backslash and keeps the letter. Redundant **slashes** in a path are collapsed by the kernel's path resolver. Both transformations happen after the filter inspects the raw bytes.

## Backslash escaping an ordinary character

Outside quotes, `\x` is treated as a quoted `x`. Since quoting a plain letter changes nothing about the letter, `w\ho\am\i` tokenizes to `whoami`:

```
w\ho\am\i
c\at /etc/passwd
i\d
```

The backslash characters are removed during quote removal, reassembling the keyword. A blocklist matching the literal `whoami` sees only the escaped form.

## Slash insertion in paths

The path resolver treats runs of `/` as a single separator and ignores `/./`, so an absolute path can be padded without changing the target file:

```
/bin/c\at /etc/passwd
//bin//cat /etc//passwd
/bin/./cat /etc/./passwd
```

`/bin/c\at` combines both tricks: the backslash splits the binary name while the explicit path still points at `cat`. This defeats filters keyed on `/bin/cat` or on the bare word `cat`.

## In an HTTP request

URL-encode the backslash as `%5c` where the application does not accept it raw:

```
w%5cho%5cam%5ci
```

## Combining with other evasions

These tricks hide the keyword or path only. Add a space substitute or separator where needed:

```
/bin/c\at${IFS}/etc/passwd
127.0.0.1;w\ho\am\i
```

## Context and caveats

- This is a **shell** technique (`sh -c`, `system()`, backticks). In a pure `argv` call the backslashes are literal bytes of the argument and are **not** stripped.
- Inside single quotes a backslash is literal, so this splitting form requires an unquoted or double-quoted context.
- Backslash-escaping a *letter* is distinct from backslash-**newline** continuation, which folds two lines together; see the dedicated continuation page.

## References

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
