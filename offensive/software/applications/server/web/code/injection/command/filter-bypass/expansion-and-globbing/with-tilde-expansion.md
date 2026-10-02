---
title: "Command injection filter bypass with tilde expansion"
description: "Rebuilding filtered paths from the shell's tilde expansion, ~ for HOME, ~+ for PWD, ~- for OLDPWD, so directory prefixes are produced by the shell rather than typed as filtered literals."
keywords:
  - command injection
  - tilde expansion
  - filter bypass
  - path reconstruction
  - PWD
  - OLDPWD
---

# Tilde expansion

Tilde expansion replaces a leading `~` with a directory path before the command runs. Plain `~` becomes `$HOME`, `~+` becomes `$PWD` (the current directory), and `~-` becomes `$OLDPWD` (the previous directory). Because the shell produces the path itself, an attacker can rebuild a filesystem prefix without typing the literal directory name a blocklist is watching for.

## Why the shell normalizes it away

Tilde expansion happens early in the shell's expansion sequence, alongside brace expansion and parameter expansion, before word splitting and before execution. A word beginning with `~`, `~+`, or `~-` is rewritten to the corresponding directory string. The filter inspecting the request sees only the tilde token; the shell hands the command a fully qualified path. The forbidden literal (for example a home-directory prefix, or a path that would otherwise contain a blocked keyword) is manufactured from shell state rather than supplied by the attacker.

## Payloads

`~+` expands to the current working directory, which is useful when a relative path is filtered or when you need an absolute prefix without typing `/`:

```
cat ~+/config/secret.env
~+/uploaded_binary
```

`~` reaches the service account's home directory, often where application config, SSH keys, or history files live, without naming it:

```
cat ~/.ssh/id_rsa
cat ~/.bash_history
ls -la ~
```

`~-` references `OLDPWD`; where an application changes directories between operations, it can point at a previously visited location:

```
cat ~-/.env
```

Named-user tilde (`~user`) expands to that account's home directory from `/etc/passwd`, letting you target another user's files without a literal path:

```
cat ~www-data/.config
ls ~root 2>/dev/null
```

Tilde combines with no-space tricks, since the technique only supplies the directory prefix and you still need separators:

```
cat${IFS}~/.ssh/id_rsa
{cat,~/.bash_history}
```

It also pairs with variable slicing to rebuild a leading slash elsewhere in the command while `~+` handles the working-directory prefix.

## Operational notes

- Tilde expansion is a feature of Bash and most interactive shells; confirm the sink spawns a shell that performs it. Expansion applies only to an **unquoted** leading `~`, inside quotes it is a literal.
- `~+`/`~-` depend on `PWD`/`OLDPWD` being set, which they normally are in a shell spawned with an environment. The value reflects wherever the application's child process is running.
- The technique rebuilds **directory prefixes**, not binary names, pair it with globbing or variable expansion when the command or the forbidden characters themselves are filtered.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting tilde-expansion payloads.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Tilde Expansion](https://www.gnu.org/software/bash/manual/html_node/Tilde-Expansion.html)
