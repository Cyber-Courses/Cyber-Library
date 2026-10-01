---
title: "JSON log injection: injecting keys into string-built structured logs"
description: "When a JSON log line is assembled by string concatenation, injecting a quote and comma to add or override keys like level, user, and trace, break the aggregator schema, or smuggle nested objects into ELK or Loki."
keywords:
  - log injection
  - JSON log injection
  - structured logging
  - ELK injection
  - Loki injection
  - schema poisoning
---

# JSON log injection

Structured logging emits one JSON object per line, consumed by aggregators such as Elasticsearch, Loki, or a cloud log service. When the JSON is produced by a real serializer, embedded quotes are escaped and the object is safe. When it is instead assembled by string concatenation, an attacker-controlled value that contains a `"` can close its string, and a following `,` can start a new key, giving the attacker control over the object's structure rather than just one field.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you own or are contracted to assess. Unauthorized use is unlawful.

## The primitive

A hand-built JSON log line looks like:

```
out.write('{"ts":"' + ts + '","level":"INFO","msg":"login for ' + user + '"}')
```

With `user` set to a benign value the line is valid. Set `user` to a value that closes the `msg` string and appends keys:

```
bob","level":"ERROR","evil":"
```

The emitted line becomes:

```
{"ts":"2026-10-01T09:14:22Z","level":"INFO","msg":"login for bob","level":"ERROR","evil":""}
```

The object now has two `level` keys and an injected `evil` key. The payload from the task statement, `","level":"INFO","x":"`, is the same shape: close the current value, add a key, and reopen a string so the trailing `"}` still closes cleanly.

## Overriding keys through last-wins parsing

JSON permits duplicate keys, and most parsers keep the last occurrence. Because the injected keys appear after the legitimate ones, they win. This lets an attacker override fields the application set:

- Spoof `level` to hide activity below a dashboard's severity filter, or raise a decoy:

```
bob","level":"DEBUG","msg":"
```

- Spoof identity so the event is attributed elsewhere:

```
bob","user":"administrator","role":"admin","msg":"
```

- Overwrite correlation fields such as `trace`, `request_id`, or `span_id` to detach the record from a real request trail or to collide with another user's trace:

```
bob","trace":"00000000000000000000000000000000","msg":"
```

## Adding fields the schema never expected

Injecting a brand-new key pollutes the index. In Elasticsearch, a new field is mapped on first sight; a value whose type conflicts with the existing mapping triggers a mapping exception and the whole document is rejected, so the real event is dropped from the index:

```
bob","status":{"nested":true},"msg":"
```

Here `status`, if previously mapped as a keyword, now arrives as an object and the aggregator refuses the line. Repeating distinct new keys can also inflate the mapping toward its field limit.

## Smuggling nested objects and arrays

Because the attacker controls raw JSON, not just a scalar, nested structure can be injected to shape how the record renders in a viewer or to carry a payload into a field an alert reads literally:

```
bob","tags":["forged","high-priority"],"geo":{"country":"CH","ip":"8.8.8.8"},"msg":"
```

In Loki, where each line is text labeled by stream, injected keys that a downstream pipeline stage parses with a `json` parser become new extracted labels or fields, letting an attacker fabricate labels the stream never legitimately carried.

## Breaking the line entirely

Where the goal is to drop a record rather than forge one, an unbalanced brace or an injected newline inside the string makes the line invalid JSON. Strict ingesters discard the malformed line, removing the activity it would have recorded:

```
bob"}%0a{"ts":"forged
```

This splits one object into a broken fragment and a second partial object, and is the JSON analogue of newline forging against plaintext logs.

## Encoding notes

- Over HTTP form or query input, send the quote and comma literally; where the sink is itself JSON, supply them as `\"` and `,` so they decode to raw structure at the logger.
- A real serializer defeats all of the above, so this technique applies specifically to string-concatenated or format-string JSON logging.

## References

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [PayloadsAllTheThings: CRLF and Log Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
