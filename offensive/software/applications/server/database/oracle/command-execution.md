---
title: "Oracle command execution: scheduler, Java, and external tables"
description: "Running operating-system commands from Oracle Database as the service account through DBMS_SCHEDULER external jobs, Java stored procedures, and external-table preprocessors, the three sinks ODAT automates once you hold a sufficiently privileged account."
keywords:
  - DBMS_SCHEDULER
  - Java stored procedure
  - external table
  - ODAT
  - Oracle RCE
---

# Oracle command execution

With a privileged account (DBA or the right `CREATE`/`EXECUTE` grants), Oracle runs operating-system commands as its **service account** through several subsystems. `odat` wraps all three; pick whichever the account's privileges and the database version allow.

## DBMS_SCHEDULER external jobs

The scheduler can run an external executable as a job:

```bash
odat dbmsscheduler -s <target> -d <SID> -U user -P pass --exec "/bin/bash -c 'id>/tmp/o'"
```

## Java stored procedures

Where Java is installed, a stored procedure wrapping `Runtime.exec` runs commands in the database's JVM:

```bash
odat java -s <target> -d <SID> -U user -P pass --exec "id"
```

## External-table preprocessor

An external table with a **preprocessor** directive runs a program when the table is read, a route that works without Java:

```bash
odat externaltable -s <target> -d <SID> -U user -P pass --exec "/path" "id"
# also: externaltable can read and write host files (--getfile / --putfile)
```

## Exploitation notes

- Commands run as the **Oracle service account** (`oracle` on Linux, often a privileged service account on Windows), so the payoff is host access as that account.
- The three sinks need different privileges: pick based on what [enumeration](access-and-enumeration.md) showed (`CREATE JOB`/`CREATE PROCEDURE`/`CREATE ANY DIRECTORY`), and let `odat` try them in turn.
- The **external-table** route doubles as a file read/write primitive, useful when you only need to drop a payload or steal a file.
- PL/SQL injection in a `DEFINER`-rights package can supply the missing privilege, turning a low account into one that reaches these sinks.

## Tools

- **ODAT** (`dbmsscheduler`, `java`, `externaltable`): automated command execution across all three sinks.
- **sqlplus**: run the PL/SQL directly where you prefer manual control.

## References

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: Oracle command execution](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: DBMS_SCHEDULER](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SCHEDULER.html)
