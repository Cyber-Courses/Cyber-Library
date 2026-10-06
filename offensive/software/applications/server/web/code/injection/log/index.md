---
title: "Log injection"
order: 10
description: "Untrusted input written into log lines or fields, letting an attacker forge events, shift fields, and confuse the parsers, aggregators, and SIEM pipelines that read the logs downstream."
keywords:
  - log injection
  - log forging
  - CRLF injection
  - log poisoning
  - SIEM evasion
---

# Log

Log injection happens when attacker-controlled input is written into a log line or a structured log field without the logger neutralizing the characters that define record boundaries, delimiters, or object structure.

Because most logging code is a thin string concatenation around whatever the request supplied, an attacker who controls a username, header, path, or query parameter often controls bytes that the log format treats as structure rather than content. The payoff is not usually code execution in the logger itself but control over the record: forging whole log entries, spoofing severity and source fields, framing another account, breaking the regex or schema that a downstream parser depends on, and smuggling markup that fires when an analyst opens the log in a web console. Logged input can also reach lookup sinks that evaluate it, which is out of scope here. This subtree is organized by the log format being broken: newline forging against line-oriented logs, delimited field injection against CSV, TSV, and `key=value` logs, and JSON log injection against structured pipelines feeding ELK or Loki.

## Pages

- **[Delimited field injection](delimited-field-injection.md)**: Injecting the delimiter or quote character of a structured log format so logged input shifts columns, adds fake fields, or breaks the regex parsers that cons...
- **[JSON log injection](json-log-injection.md)**: When a JSON log line is assembled by string concatenation, injecting a quote and comma to add or override keys like level, user, and trace, break the aggrega...
- **[Newline forging](newline-forging.md)**: Injecting CRLF or a bare newline into logged input so a single log statement emits two lines, forging a second log record with attacker-chosen timestamp, sev...

## Tools

- **Burp Repeater**: crafting CRLF, delimiter, and JSON field payloads into logged input.
- **curl**: sending crafted log-injection payloads to request parameters.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
