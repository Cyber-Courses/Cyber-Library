---
title: "MSSQL access: authenticating to the instance"
order: 2
description: "Getting an authenticated SQL Server session: SQL logins through default and weak credentials or spraying, Windows and domain authentication, and the role of the public role and guest access as the starting foothold."
keywords:
  - MSSQL access
  - SQL login
  - Windows authentication
  - password spraying
  - public role
---

# Access

SQL Server accepts two authentication modes, and both are routes in: **SQL logins** (username and password stored in the instance) and **Windows authentication** (local or domain accounts). The goal here is any authenticated session, however low-privileged, because `public` alone is enough to begin enumeration and often coercion.

## SQL logins

Default and weak SQL logins are common, above all `sa`:

```bash
# Authenticate a SQL login
mssqlclient.py 'sa:Password123!@<target>'
impacket-mssqlclient WORKGROUP/sa:'Password123!'@<target>

# Spray SQL logins across an instance (respect lockout where set)
nxc mssql <target> -u sa -p passwords.txt --local-auth
```

## Windows and domain authentication

```bash
# Domain account (password or hash), mapping to a Windows login/principal
mssqlclient.py -windows-auth 'example.local/user:password@<target>'
nxc mssql <target> -u user -p password -d example.local
nxc mssql <target> -u user -H <nthash> -d example.local   # pass-the-hash to MSSQL
```

## Exploitation notes

- `sa` is **sysadmin**, so an `sa` hit is immediate [command execution](command-execution.md); a weak or reused `sa` password is the classic MSSQL win.
- A domain login that maps into the instance is often more useful than a SQL login, because it can be reached by [relay](coercion-and-relay.md) and by [pass-the-hash](../../directory/active-directory/authentication/ntlm/pass-the-hash.md) to MSSQL.
- Even an unprivileged `public` session enables [enumeration](enumeration.md), the coercion procedures, and a search for [impersonation](impersonation.md) and [linked-server](linked-servers.md) paths.
- `guest` access to a database widens reach without an explicit grant, so check which databases a low login can `USE`.

## Tools

- **Impacket `mssqlclient.py`** (`-windows-auth`, `-hashes`): interactive sessions with SQL or Windows auth and pass-the-hash.
- **NetExec `mssql`** (`-u/-p/-H`, `--local-auth`): authentication, spraying, and pass-the-hash at scale.

## References

- [NetExec: MSSQL authentication](https://www.netexec.wiki/mssql-protocol/authentication)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [Microsoft: choose an authentication mode](https://learn.microsoft.com/en-us/sql/relational-databases/security/choose-an-authentication-mode)
