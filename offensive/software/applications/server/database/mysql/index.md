---
title: "MySQL and MariaDB"
description: "The offensive surface of MySQL and MariaDB reached as a service: authenticating, writing host files through INTO OUTFILE and reading them with LOAD_FILE, and executing operating-system commands through user-defined functions."
keywords:
  - MySQL
  - MariaDB
  - INTO OUTFILE
  - user-defined function
  - FILE privilege
---

# MySQL and MariaDB

MySQL and its fork MariaDB (port 3306) give two main offensive primitives once you authenticate: **file read and write** through `LOAD_FILE` and `INTO OUTFILE` (gated by the `FILE` privilege and `secure_file_priv`), and **operating-system command execution** through a user-defined function. The file write alone is often enough, dropping a webshell under a served directory.

## What to reach for

- **[Command execution](command-execution.md)**: a user-defined function (`lib_mysqludf_sys`) that runs OS commands as the service account.
- **[File access](file-access.md)**: `INTO OUTFILE` to write a webshell, `LOAD_FILE` to read host files.

## Getting a session

```bash
# default/weak credentials, commonly root with no or a weak password
mysql -h <target> -u root -p
hydra -L users.txt -P passwords.txt <target> mysql
```

## References

- [HackTricks: pentesting MySQL (3306)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
- [MySQL: the FILE privilege and secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_file)
