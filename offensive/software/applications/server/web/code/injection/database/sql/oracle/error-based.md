---
title: "Error-based SQL injection in Oracle"
description: "Leaking Oracle query results through ORA errors that echo attacker input, using CTXSYS.DRITHSX.SN and UTL_INADDR, with version and ACL caveats."
keywords:
  - error based SQL injection
  - CTXSYS.DRITHSX.SN
  - UTL_INADDR
  - ORA-20000
  - ORA-29257
---

# Error-based

When the application hides rows but reflects database errors, Oracle leaks data through functions that raise an error containing their argument. Two are reliable, each with caveats.

`CTXSYS.DRITHSX.SN` raises `ORA-20000` with the offending text and is often callable without special network grants (it needs the Oracle Text component, present in most installs):

```sql
' AND 1=CTXSYS.DRITHSX.SN(1,(SELECT user FROM dual))-- 
```

The error reads `ORA-20000: Oracle Text error: DRG-11701: thesaurus SCOTT does not exist` (with the value in place of the thesaurus name), leaking the subquery result.

`UTL_INADDR.GET_HOST_NAME` raises `ORA-29257: host <value> unknown`, echoing the string when name resolution fails:

```sql
' AND 1=UTL_INADDR.GET_HOST_NAME((SELECT banner FROM v$version WHERE ROWNUM=1))-- 
```

This one is network-related, so on 11g and later it is gated by a fine-grained ACL on `UTL_INADDR`; without the ACL grant it raises an access error instead of the leaking one. Other channels named in payload lists (`XMLType` parse errors, `DBMS_XMLGEN`, `ORD_DICOM.GETMAPPINGXPATH`) vary by version and installed components.

Keep the inner query to a single value with `ROWNUM=1` or an aggregate. Because these raise distinct ORA codes on success, the same functions also drive blind error-based extraction when only the presence of the error, not its text, is observable.

## Tools

- **sqlmap**: automated error-based extraction with Oracle payloads (`--dbms=Oracle --technique=E`).
- **ghauri**: fast alternative with strong WAF evasion.
- **sqlplus** (or SQLcl): the Oracle client for replaying and refining the error-raising functions.

## References

- Oracle Database PL/SQL Packages and Types Reference: UTL_INADDR; Oracle Text Reference
- OWASP Testing Guide: Testing for SQL Injection
