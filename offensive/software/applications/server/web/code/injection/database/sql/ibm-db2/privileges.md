---
title: "Enumerating authorities and privileges in IBM Db2 injection"
order: 10
description: "Reading the current IBM Db2 authorization ID's authorities and grants from SYSCAT.DBAUTH to decide which primitives, including routine creation, are reachable."
keywords:
  - Db2 privileges
  - SYSCAT.DBAUTH
  - DBADM
  - SYSADM
  - authorization ID
---

# Privileges

The authorities held by the current authorization ID decide how far an injection goes, so privilege enumeration follows fingerprinting. Database-level authorities are in `SYSCAT.DBAUTH`. Authority can be granted to the user directly, to a group or role it belongs to, or to `PUBLIC`, so filtering on the user alone misses inherited grants; include those grantees:

```sql
' UNION SELECT GRANTEE,GRANTEETYPE,DBADMAUTH FROM SYSCAT.DBAUTH WHERE GRANTEE=CURRENT USER OR GRANTEE='PUBLIC' OR GRANTEETYPE='R'-- 
```

The effective-authority routine `SYSPROC.AUTH_LIST_AUTHORITIES_FOR_AUTHID(CURRENT USER,'U')` resolves the full set including role and group inheritance, which is the reliable check. `DBADMAUTH='Y'` means the ID holds `DBADM` (database administration), which is broad. The instance-level `SYSADM`, `SYSCTRL`, and `SYSMAINT` authorities come from the database manager configuration rather than the catalog, and `SECADM` governs security objects. Object grants are in `SYSCAT.TABAUTH` (table privileges) and `SYSCAT.ROUTINEAUTH` (execute on routines):

```sql
' UNION SELECT TABSCHEMA,TABNAME,SELECTAUTH FROM SYSCAT.TABAUTH WHERE GRANTEE=CURRENT USER OR GRANTEE='PUBLIC' OR GRANTEETYPE='R'-- 
```

The authorities that matter for escalation are those allowing routine creation and execution. `CREATE_EXTERNAL_ROUTINE` (and `DBADM`) let the ID create external C or Java routines, which is the Db2 path to command execution; `IMPLICIT_SCHEMA` and broad `GRANT` rights help plant objects. An ID with `DBADM` or `SYSADM` can also grant itself further authorities.

Knowing the authorities tells you whether to pursue command execution through an external routine, stay with data extraction, or first escalate by granting missing authorities where the current ID is permitted to. The special register `CURRENT USER` identifies whose grants these are.

## Tools

- **sqlmap**: enumerates the current user's privileges (`--privileges`) against Db2.
- **db2** (or clpplus): the Db2 client for reading `SYSCAT.DBAUTH` and `AUTH_LIST_AUTHORITIES_FOR_AUTHID`.
- Manual testing with Burp Repeater and the Db2 client.

## References

- IBM Db2 SQL Reference and Administration Guide: authorities, SYSCAT.DBAUTH, SYSCAT.ROUTINEAUTH
- OWASP Testing Guide: Testing for SQL Injection
