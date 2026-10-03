---
title: "Oracle Database"
description: "The offensive surface of Oracle Database reached through the TNS listener: enumerating SIDs and services, testing default and weak accounts, and executing operating-system commands through Java, the scheduler, and external tables."
keywords:
  - Oracle
  - TNS listener
  - SID
  - ODAT
  - default credentials
---

# Oracle Database

Oracle Database is reached through the **TNS listener** (port 1521), which routes a client to a database instance named by a **SID** or service name. The attack path is staged: enumerate the listener and valid SIDs, test accounts (Oracle is notorious for **default credentials**), then, with a database account, reach operating-system command execution through Java, the scheduler, or external tables. `odat` automates most of it.

## What to reach for

- **[Access and enumeration](access-and-enumeration.md)**: TNS and SID enumeration, default and weak accounts, schema and hash extraction.
- **[Command execution](command-execution.md)**: OS commands through `DBMS_SCHEDULER`, Java stored procedures, and external tables.

## References

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: pentesting Oracle TNS (1521)](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
