---
title: "Newline injection: using a line return as a command terminator in sh -c"
description: "Injecting a raw or URL-encoded newline (%0a, \\n) to terminate the intended command and start a new one inside sh -c, bypassing filters that only block ; | &."
keywords:
  - command injection
  - newline injection
  - line feed
  - "%0a"
  - filter bypass
  - sh -c
---

# Newline as a command terminator

Filters frequently blocklist the obvious shell separators, `;`, `|`, `&`, while overlooking that a **newline is itself a command terminator**. Inside `sh -c "..."`, every line of the string is parsed as its own command, exactly as in a script file. A single injected line feed ends the intended command and begins an attacker-controlled one, with no banned metacharacter in sight.

## Why it works

When a shell reads `ping -c 1 HOST`, the grammar treats a newline the same way it treats `;`: it completes the current simple command. If the attacker's value becomes `127.0.0.1\nwhoami`, the shell executes `ping -c 1 127.0.0.1`, then, on the next line, `whoami`. Because `\n` is control structure rather than a "special character" in the filter author's mental model, it is routinely missed.

## Payloads

In an HTTP request the newline is URL-encoded. The line feed is `%0a`; some stacks require or also accept a carriage return `%0d`:

```
127.0.0.1%0awhoami
127.0.0.1%0aid
127.0.0.1%0d%0awhoami
```

As a raw byte (JSON body, header, or a terminal paste) it is a literal newline:

```
127.0.0.1
whoami
```

In language sinks the escape sequence must survive into the final string before the shell sees it:

```
host = "127.0.0.1\nid"      # when the runtime unescapes before sh -c
```

## Pairing with other tricks

The newline only supplies the separator. If the command keyword is also filtered, combine it with keyword splitting or an `${IFS}` space substitute:

```
127.0.0.1%0awho${IFS}ami
127.0.0.1%0a/???/??t${IFS}/etc/passwd
```

## Context notes

- The injected string must land in a **shell** context (`sh -c`, `system()`, backticks). A pure `argv` call does not reinterpret the newline as a separator.
- `cmd.exe` does not treat a bare line feed as a separator the same way; prefer `&` there. Newline termination is a POSIX-shell technique.
- Where the response reflects nothing, confirm with a timing or out-of-band second command on the new line.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
