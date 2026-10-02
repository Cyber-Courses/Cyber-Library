---
title: "Time-based blind SQL injection in MySQL"
description: "Inferring MySQL data from conditional response delays using SLEEP and BENCHMARK when the response is otherwise identical for true and false."
keywords:
  - time based blind
  - SLEEP injection
  - BENCHMARK
  - conditional delay
  - MySQL blind timing
---

# Time-based

When true and false conditions produce the same response, inference falls back on time. A payload that delays the response only when a condition is true turns the response latency into the yes/no signal: a slow answer means true, a prompt answer means false.

MySQL delays with `SLEEP(seconds)`, which pauses the current row's evaluation, and with `BENCHMARK(count, expr)`, which burns CPU by repeating an expression. The delay must be made conditional so that it fires only on a true test, which is what carries one bit of information per request.

The key dialect detail is that the delay needs a row context or an expression context. `SELECT SLEEP(5) WHERE <test>` is a syntax error in MySQL because it allows a `SELECT` with no `FROM` only for constants, and a `WHERE` without a `FROM` is invalid (unlike PostgreSQL and SQLite). The delay is therefore placed inside `IF()` or attached to a row source instead. The pages below show both correct forms.

Timing is noisier than a boolean oracle because network jitter and server load affect latency, so delays are kept several seconds long and confirmed with a repeat before a bit is trusted.

## Pages

- **[SLEEP](sleep.md)**: conditional `IF(..., SLEEP(n), 0)` extraction.
- **[BENCHMARK](benchmark.md)**: CPU-based delay when `SLEEP` is filtered.

## Tools

- **sqlmap**: automated time-based blind extraction with SLEEP and BENCHMARK.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- MySQL Reference Manual: SLEEP, BENCHMARK, IF
- PortSwigger Web Security Academy: Blind SQL injection
