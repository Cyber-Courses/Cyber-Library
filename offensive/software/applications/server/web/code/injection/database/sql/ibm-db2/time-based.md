---
title: "Time-based blind SQL injection in IBM Db2"
description: "Inferring IBM Db2 data from conditional delays built with a heavy query, since Db2 has no SLEEP or WAITFOR function."
keywords:
  - time based blind
  - heavy query
  - no SLEEP
  - cross join
  - Db2 timing
---

# Time-based

Db2 has no `SLEEP` and no `WAITFOR` (`WAITFOR` is T-SQL, a common mislabel), so a delay is produced by forcing the database to do expensive work only when a condition is true. A large cross join over a catalog view is the usual device.

Joining a big catalog view to itself several times produces a huge intermediate result, and counting it costs measurable CPU. Gate it with `CASE` so the cost is paid only on a true test:

```sql
' AND (SELECT CASE WHEN (ASCII(SUBSTR((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1),1,1))>77) THEN (SELECT COUNT(*) FROM SYSCAT.COLUMNS a, SYSCAT.COLUMNS b, SYSCAT.COLUMNS c) ELSE 1 END FROM SYSIBM.SYSDUMMY1)>0-- 
```

When the test is true, Db2 evaluates the triple cross join (often seconds on a normal catalog); when false, the `ELSE` returns immediately. Tune the number of joined copies, or use a larger view such as `SYSIBM.SYSCOLUMNS`, so a true test adds a clear, repeatable delay without overloading the server.

Binary-search each character on the delay exactly as in boolean extraction. Because the delay is CPU-bound rather than a fixed sleep, it is noisier than `pg_sleep` or `WAITFOR`, so keep the work large enough to read clearly and repeat a positive hit before trusting it. Given Db2's weak error channel, boolean inference is usually preferred, with this heavy-query timing as the fallback when no boolean difference is visible.

## Tools

- **sqlmap**: automated time-based extraction with heavy cross-join payloads (`--technique=T`).
- **ghauri**: fast time-based inference with strong WAF evasion.
- **Burp Repeater**: measure the conditional delay by hand to tune the join count.

## References

- IBM Db2 SQL Reference: subselect, joins, CASE, catalog views
- PortSwigger Web Security Academy: Blind SQL injection
