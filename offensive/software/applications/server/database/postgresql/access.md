---
title: "PostgreSQL access: authenticating to the cluster"
description: "Reaching an authenticated PostgreSQL session through default and weak credentials, trust authentication that needs no password, and the roles that matter, as the starting point for command execution and file access."
keywords:
  - PostgreSQL access
  - trust authentication
  - pg_hba.conf
  - postgres superuser
  - psql
---

# PostgreSQL access

The target is any authenticated session, ideally as a **superuser** role. The default administrative role is `postgres`, and misconfigured `pg_hba.conf` entries frequently make access trivial.

## Getting a session

```bash
# Default/weak credentials for the postgres superuser
psql "host=<target> user=postgres password=postgres dbname=postgres"

# Spray credentials
hydra -L users.txt -P passwords.txt <target> postgres
```

## Trust authentication

`pg_hba.conf` can map a host or network to the **`trust`** method, which accepts any username **with no password**. Where the service is exposed and a `trust` line covers your source, you log in as any role, including `postgres`, for free:

```bash
psql "host=<target> user=postgres dbname=postgres"   # no password prompt under trust
```

## Exploitation notes

- Check your role's privileges immediately: `SELECT current_user, usesuper FROM pg_user WHERE usename = current_user;`. A superuser goes straight to [command execution](command-execution.md).
- A non-superuser is still useful: certain roles (`pg_read_server_files`, `pg_execute_server_program`) grant [file](file-access.md) and program access without full superuser, and several paths escalate an ordinary role to superuser.
- `trust` on an internet-facing cluster is an unauthenticated superuser foothold, so always test it before spraying.

## Tools

- **psql**: the native client for interactive access and testing.
- **hydra / metasploit `postgres_login`**: credential spraying.

## References

- [PostgreSQL: pg_hba.conf authentication methods](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
- [HackTricks: pentesting PostgreSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
