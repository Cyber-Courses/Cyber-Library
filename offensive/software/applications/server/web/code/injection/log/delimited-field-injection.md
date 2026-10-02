---
title: "Delimited field injection: breaking CSV, TSV, and key=value log columns"
description: "Injecting the delimiter or quote character of a structured log format so logged input shifts columns, adds fake fields, or breaks the regex parsers that consume CSV, TSV, and key=value logs downstream."
keywords:
  - log injection
  - CSV injection
  - delimiter injection
  - key=value logs
  - log parser evasion
  - field injection
---

# Delimited field injection

Many logs are not free text but delimited records: comma- or tab-separated columns, or a run of `key=value` pairs. A parser downstream splits each line on the delimiter and assigns meaning by position or by key. When attacker input is written into one field without escaping the delimiter or quote character, the input stops being a single value and becomes extra fields, shifting every column after it and letting the attacker set values the application never intended to log.

## Shifting columns in CSV and TSV logs

Consider a CSV audit log where columns are `timestamp,user,action,result,source_ip`:

```
2026-10-01T09:14:22Z,bob,login,fail,203.0.113.9
```

If `user` is attacker-controlled and commas are not escaped or the value is not quoted, a comma in the username injects new columns:

```
bob,login,success,10.0.0.5
```

The line becomes:

```
2026-10-01T09:14:22Z,bob,login,success,10.0.0.5,login,fail,203.0.113.9
```

A position-based parser now reads `action=login`, `result=success`, `source_ip=10.0.0.5` from the forged fields and treats the real trailing columns as overflow. The recorded outcome is inverted.

TSV logs break the same way with a tab, sent as `%09` over HTTP:

```
bob%09login%09success%0910.0.0.5
```

## Breaking out of a quoted field

Well-formed CSV wraps values containing a delimiter in double quotes and doubles internal quotes. If the writer quotes but does not double the attacker's quotes, a `"` closes the field early and a following comma starts a new column:

```
bob","admin","success
```

Against `...,"<user>",...` this yields three fields where one was intended, with `admin` landing in the next column. A trailing unmatched quote can also swallow the rest of the line into one field, hiding genuine columns from a strict parser.

## Injecting and overriding key=value fields

Structured text logs often use space-separated `key=value` pairs:

```
ts=2026-10-01T09:14:22Z user=bob action=login result=fail ip=203.0.113.9
```

A space and an `=` in the `user` value inject additional keys. Whether an injected key overrides the real one depends on the parser's duplicate-key rule (first-wins vs last-wins) and where the injected field sits relative to the genuine one:

```
bob result=success role=admin
```

The line becomes:

```
ts=... user=bob result=success role=admin action=login result=fail ip=...
```

Because the attacker-controlled `user` field precedes `result`, the injected `result=success` sits *before* the genuine `result=fail`. A **first-wins** parser keeps the first occurrence, so it resolves `result=success` (and `role=admin` appears from nowhere); that is the case this example exploits. A **last-wins** parser keeps the trailing genuine `result=fail`, so to beat it the attacker must instead control a field written *after* the real key, typically the final field on the line.

## Breaking downstream regex parsers

Log shippers (Logstash grok, Fluentd, custom regex) assume a fixed field count and character class per column. Input that contains the delimiter, an unbalanced quote, or a newline can make a line fail the pattern. A failed line is commonly routed to a dead-letter or `_grokparsefailure` path and dropped from the indexed stream, so activity logged on that line never reaches the dashboards defenders watch:

```
bob"," unterminated field that defeats the column regex
```

Flooding a field with the delimiter can also force the parser into its slow backtracking path, delaying ingestion of the records around it.

## Encoding notes

- Comma is literal in most contexts; tab is `%09`, and a field-splitting space is literal or `%20` depending on the surrounding format.
- Combine with a newline (`%0a`) to both break the current line's parse and forge a fresh record, since many delimited parsers are also line-oriented.

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
