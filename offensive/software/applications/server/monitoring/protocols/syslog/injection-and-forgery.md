---
title: "Injection and forgery: forged entries and downstream parser abuse"
order: 2
description: "Syslog ingestion trusts message content, so an attacker injects forged entries and, by embedding newlines and delimiters, splits one record into several or breaks field parsing, poisoning SIEM correlation. Where the pipeline feeds logged values into a vulnerable sink, a dashboard, a parser, or a downstream app, injected content also exploits that sink."
keywords:
  - log injection
  - newline injection
  - forgery
  - siem poisoning
  - log4shell
---

# Injection and forgery

Because the syslog pipeline trusts message content, an attacker who can emit records (directly to the collector, or indirectly by causing a monitored application to log attacker-controlled data) forges and manipulates the record. Straight forgery plants false events. More subtly, embedding newline and delimiter characters in a logged value splits one record into multiple apparent events or breaks the collector's field parsing, so an attacker can fabricate extra log lines, desynchronise structured parsing, and poison the correlation rules and dashboards that consume the stream. And where logged values flow into a vulnerable sink, a log-viewer that renders them, a parser that evaluates them, or a downstream application, the injected content exploits that sink; the Log4Shell class, where a logged string triggered a JNDI lookup, is the extreme example of attacker data in a log becoming code execution.

```bash
# forge an event, and split a record with an embedded newline to fake extra lines
printf '<34>app: user=admin action=login\n<34>app: CRITICAL all clear\n' \
  | nc -u -w1 <collector> 514
# indirect: make a monitored app log attacker-controlled data that carries
# newlines/delimiters (e.g. a crafted username, User-Agent, or URL) to inject
# downstream; if the sink evaluates logged values, injection reaches that sink
```

## Exploitation notes

- Newline/CRLF and delimiter injection is the core technique: it turns one logged value into several records or corrupts structured-field parsing, letting an attacker fabricate events and desynchronise the SIEM without ever spoofing a packet.
- Indirect injection is often the realistic vector: attacker-controlled fields that applications log (usernames, headers, URLs, error messages) carry the payload, so the attacker never needs to speak syslog directly.
- The highest impact is a vulnerable downstream sink: if logged values are rendered, parsed, or evaluated unsafely, injection becomes stored XSS in a log dashboard, parser exploitation, or code execution (the Log4Shell pattern); assess what consumes the logs.
- Pair with [source spoofing](source-spoofing.md) to control origin and content together for convincing forged events.

## References

- [OWASP: log injection](https://owasp.org/www-community/attacks/Log_Injection)
- [RFC 5424 (syslog message format)](https://datatracker.ietf.org/doc/html/rfc5424)
