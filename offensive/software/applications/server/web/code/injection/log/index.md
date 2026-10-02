---
title: "Log injection: forging log lines and corrupting the record attackers are judged by"
description: How untrusted input written into application logs forges fake entries, breaks structured-log parsers, and reaches downstream sinks—log viewers, SIEM pipelines, and shell automation—that trust log content as if it were ground truth.
keywords:
  - log injection
  - log forging
  - newline injection
  - CRLF log injection
  - structured log injection
  - log4j
---

# Log injection

**Log injection** is a vulnerability in which application code writes untrusted input into a log stream without neutralizing the characters that give a log its *structure*. The log is not just a text file—it is a data format with record boundaries, field delimiters, and, increasingly, a JSON schema that other software parses mechanically. When an attacker controls bytes inside a logged value, they control part of that format, and the record stops being a faithful account of what happened.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Tampering with logs on systems you are not permitted to test is unlawful and destroys evidence others rely on.

## Overview

Applications log for the most ordinary reasons: a failed login, a request line, a username, a user-agent string, the value that triggered an error. The text handed to `logger.info(...)` is assembled by concatenation, and the untrusted parts of it—headers, form fields, usernames—arrive with whatever bytes the attacker chose. Two distinct mechanisms turn that into an exploitable bug, and they map to the two pages below.

1. **Newline forging in line-oriented logs.** A plain-text log treats a newline as the end of one record and the start of the next. A single logged field that contains `\n` (or `\r\n`) becomes **two or more** log lines, each of which an operator—or a `grep`/`awk` one-liner, or a log-shipping agent—reads as a separate, real event. The attacker writes their own history. See [Newline forging in text logs](newline-forging-in-text-logs.md).

2. **Structured log-field injection.** JSON Lines, logfmt, and key-value logs carry their structure in delimiters (`"`, `{`, `}`, `=`, spaces). A value that smuggles those delimiters can terminate a record early, inject synthetic keys that override `level`/`msg`/`user`, or desynchronize the parser so a whole batch of events is misread. See [Structured log-field injection](structured-log-field-injection.md).

The decisive question when assessing any log sink is the same one that governs every injection class: *does untrusted input cross from data into the structural grammar of the format, and what downstream consumer re-parses that grammar?*

## Why it reaches the log unescaped

- **Logging feels like output, not a sink.** Developers escape data bound for HTML or SQL but treat the log as a private scratchpad, so raw concatenation (`log.info("login failed for " + user)`) is the norm.
- **Framework defaults append the message verbatim.** Classic appenders write the message string with no per-field encoding; structure lives only in the pattern layout, which the message can break out of.
- **The value is assumed benign.** A username, a referrer, a filename—"just a string"—is logged without the author picturing a newline or a quote inside it.
- **Second-order flow.** A value stored earlier (a profile field, a saved filename) is logged much later, far from where it entered, so the taint is invisible at the logging call.

## Why the log is worth attacking

A log is read by more than a human. The attacker's real target is usually a **downstream consumer** that trusts the log as authoritative:

- **Log viewers and dashboards** render entries in a browser. If the viewer emits logged text into HTML without encoding, a forged field carrying `<script>` becomes **stored XSS** that fires in the console of whoever reviews the logs—often an administrator.
- **SIEM and detection pipelines** parse fields to drive alerts. Forged `level`/`status` fields and desynchronized records let an attacker suppress the signal of their own activity or bury it under noise, and broken records can crash or stall a brittle parser.
- **Evaluating sinks.** Some logging stacks treat the logged string as more than text—historically, message interpolation and lookup features (the log4j `${jndi:...}` class of bug) evaluated attacker-controlled markup inside the logged value itself, turning a log write into remote code execution. These are version- and library-specific; triage against the exact logging dependency and release in use.
- **Shell and automation.** Operators pipe logs through ad-hoc `grep | cut | xargs`, and agents tail files into other systems; a forged line can inject fields those scripts act on.

## Pages

| Page | Focus |
|------|--------|
| [Newline forging in text logs](newline-forging-in-text-logs.md) | Injecting `\n`/`\r\n` and control characters to forge whole log lines, spoof severity prefixes, hide real events, and reach log-viewer XSS |
| [Structured log-field injection](structured-log-field-injection.md) | Breaking JSONL/logfmt grammar: early record termination, synthetic/duplicate keys, parser desync, and SIEM-field spoofing |

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [CWE-117: Improper Output Neutralization for Logs](https://cwe.mitre.org/data/definitions/117.html)
- [CWE-93: Improper Neutralization of CRLF Sequences (CRLF Injection)](https://cwe.mitre.org/data/definitions/93.html)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
