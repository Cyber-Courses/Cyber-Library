---
title: "Command execution through Oracle injection"
description: "Reaching OS command execution from a privileged Oracle injection via Java stored procedures and DBMS_SCHEDULER external jobs."
keywords:
  - Oracle command execution
  - Java stored procedure
  - DBMS_SCHEDULER
  - external job
  - DBMS_JAVA
---

# Command execution

Oracle has no single command function like SQL Server's `xp_cmdshell`, so OS execution comes from the database's richer runtimes, and every route needs high privilege (effectively DBA or specific dangerous grants), usually reached first through the privilege-escalation step.

The Java route runs native code. With `java.lang.RuntimePermission` granted, a loaded Java class can call `Runtime.getRuntime().exec()`, wrapped in a PL/SQL function and called from SQL:

```sql
SELECT DBMS_JAVA.RUNJAVA('oracle/aurora/util/Wrapper /bin/sh -c id') FROM dual;
```

or, more commonly, by loading a small Java source that exposes an `exec` method and publishing it as a PL/SQL function. This requires the Java permissions and the ability to create Java sources, both DBA-tier.

The `DBMS_SCHEDULER` route runs an external program directly. An `EXECUTABLE` job needs both `CREATE JOB` and `CREATE EXTERNAL JOB` (not either one alone), or `CREATE ANY JOB` to create it in another schema. Create a job whose `job_type` is `EXECUTABLE`:

```sql
BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name=>'x',job_type=>'EXECUTABLE',job_action=>'/bin/sh',number_of_arguments=>2,enabled=>FALSE); ... DBMS_SCHEDULER.ENABLE('x'); END;
```

The job runs as the OS account configured for external jobs. Older Oracle also offered `DBMS_SCHEDULER` and the legacy external-procedure (`extproc`) and `PL/SQL` native routes.

All of these need an injectable PL/SQL context (Oracle does not stack plain SQL statements) and DBA-level privilege, so they are the final step after escalation. Commands run with the Oracle server's OS privileges, so that account determines the foothold.

## Tools

- **ODAT** (Oracle Database Attacking Tool): automates Java and `DBMS_SCHEDULER` external-job command execution.
- **sqlplus** (or SQLcl): the Oracle client for defining and invoking the external routine.
- Manual testing with Burp Repeater and the Oracle client.

## References

- Oracle Database Java Developer's Guide; PL/SQL Packages and Types Reference: DBMS_SCHEDULER
- OWASP Testing Guide: Testing for SQL Injection
