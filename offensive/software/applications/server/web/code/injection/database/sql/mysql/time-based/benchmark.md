---
title: "MySQL time-based injection with BENCHMARK"
description: "Using BENCHMARK to create CPU-bound delays for MySQL blind SQL injection when SLEEP is filtered, and why it is less reliable than SLEEP."
keywords:
  - BENCHMARK injection
  - CPU delay
  - SLEEP filtered
  - MySQL time based
---

# BENCHMARK

`BENCHMARK(count, expr)` evaluates `expr` `count` times and returns 0. It produces no useful value, but running an expensive expression enough times takes measurable wall-clock time, which gives a delay oracle when `SLEEP` is blocked by a filter.

Make it conditional with `IF()` so the CPU is only burned on a true test. Pair it with a deliberately costly expression such as a hash so each iteration does real work:

```sql
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77,BENCHMARK(5000000,SHA1(RAND())),0)-- 
```

Unlike `SLEEP`, the delay is not a fixed number of seconds; it depends on server speed and the iteration count, so the count must be tuned until a true test adds a clear, repeatable delay (often a few million iterations for a second or two). This makes `BENCHMARK` noisier and slower to calibrate than `SLEEP`, so it is a fallback rather than a first choice.

Because it holds a CPU core busy for every probe, high iteration counts across many characters put real load on the server. Keep the count only as high as needed for a distinguishable delay.

## Tools

- **sqlmap**: automated time-based extraction, falling back to BENCHMARK when SLEEP is filtered.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- MySQL Reference Manual: BENCHMARK, SHA1, IF
- PortSwigger Web Security Academy: SQL injection cheat sheet
