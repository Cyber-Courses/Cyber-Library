---
title: "Time-based blind SQL injection in Oracle"
order: 15
description: "Inferring Oracle data from conditional response delays with the inline DBMS_PIPE.RECEIVE_MESSAGE function, since DBMS_LOCK.SLEEP is a procedure usable only in PL/SQL."
keywords:
  - time based blind
  - DBMS_PIPE.RECEIVE_MESSAGE
  - DBMS_LOCK.SLEEP
  - conditional delay
  - Oracle timing
---

# Time-based

When true and false look identical, timing carries the signal. The key Oracle detail is which delay primitive can be called inside a `SELECT`.

`DBMS_LOCK.SLEEP` is a procedure, so it cannot be used inside a SQL expression (it runs only in a PL/SQL block, which standard drivers do not give you through a simple injection) and it also needs an explicit execute grant. The function that works inline is `DBMS_PIPE.RECEIVE_MESSAGE(pipe, timeout)`, which blocks for `timeout` seconds waiting on a (nonexistent) pipe and returns a number, so it can sit in a `CASE`:

```sql
' AND 1=(CASE WHEN (ASCII(SUBSTR((SELECT user FROM dual),1,1))>77) THEN DBMS_PIPE.RECEIVE_MESSAGE('a',5) ELSE 1 END)-- 
```

When the test is true, `RECEIVE_MESSAGE` waits five seconds before returning; when false, the `ELSE` returns immediately. Binary-search each character on the delay exactly as in boolean extraction. Execute on `DBMS_PIPE` is commonly granted to `PUBLIC`, which is why this is the default Oracle timing primitive.

Where `DBMS_PIPE` is not executable, a heavy query gives a coarse delay: a large cartesian join (for example `(SELECT COUNT(*) FROM all_objects a, all_objects b, all_objects c)`) burns measurable time, gated by a `CASE`, though it is far less precise than `RECEIVE_MESSAGE`. Keep delays several seconds long and repeat a positive hit before trusting it.

## Tools

- **sqlmap**: automated time-based extraction with Oracle `DBMS_PIPE.RECEIVE_MESSAGE` payloads (`--technique=T`).
- **ghauri**: fast time-based inference with strong WAF evasion.
- **Burp Repeater**: measure the conditional delay by hand to confirm the primitive.

## References

- Oracle Database PL/SQL Packages and Types Reference: DBMS_PIPE, DBMS_LOCK
- PortSwigger Web Security Academy: Blind SQL injection
