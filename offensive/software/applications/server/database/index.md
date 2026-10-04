---
title: "Database"
description: "Offensive scope for database servers reached as a service: authenticating, enumerating, and escalating inside the engine, executing operating-system commands from it, reading and writing host files, and pivoting across trust links, with Microsoft SQL Server as the AD-integrated flagship."
keywords:
  - database
  - MSSQL
  - PostgreSQL
  - MySQL
  - Oracle
---

# Database

A database server is reached as a network service, and once you can authenticate it is far more than a data store: the major engines can **run operating-system commands**, **read and write files on the host**, **impersonate other database principals**, and **pivot across trust links** to other servers. In a Windows estate the database service also runs as a privileged account and speaks NTLM, so it is a first-class foothold and lateral-movement target, not just a source of data.

This area organises attacks per engine, because the query language, privilege model, and command-execution primitives differ sharply between them.

## What to reach for

- **Access and enumeration**: get a login (default, weak, or captured), then map databases, roles, and the privileges you hold.
- **Command execution**: turn query access into code on the host.
- **File access**: read and write host files through the engine.
- **Privilege escalation inside the engine**: impersonation and ownership chains to reach administrative rights.
- **Lateral movement**: trust links between servers, and coercing the service account's authentication for relay.

## Engines

- **[MSSQL](mssql/index.md)**: Microsoft SQL Server, the richest and most AD-integrated target: xp_cmdshell, linked servers, impersonation, and coercion to NTLM relay.
- **[PostgreSQL](postgresql/index.md)**: superuser command execution through COPY FROM PROGRAM and untrusted languages, plus file read and write.
- **[MySQL and MariaDB](mysql/index.md)**: file write to a webshell with INTO OUTFILE and OS command execution through a user-defined function.
- **[Oracle Database](oracle/index.md)**: TNS and SID enumeration, default accounts, and command execution through the scheduler, Java, and external tables.
- **[Redis](redis.md)**: unauthenticated access and the file-write and module-load paths to code execution.
- **[MongoDB](mongodb.md)**: unauthenticated exposure, enumeration, and server-side JavaScript where enabled.

## References

- [HackTricks: pentesting MSSQL (1433)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [PayloadsAllTheThings: SQL injection and database notes](https://github.com/swisskyrepo/PayloadsAllTheThings)
