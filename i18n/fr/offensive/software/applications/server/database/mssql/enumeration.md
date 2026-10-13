---
order: 1
title: "Énumération MSSQL : instances, connexions et privilèges"
description: "Découvrir les instances et versions de SQL Server, énumérer les connexions, bases et rôles, et établir exactement quels privilèges et droits d'usurpation détient le principal courant avant de tenter l'exécution ou l'escalade."
keywords:
  - énumération MSSQL
  - SPN
  - sysadmin
  - rôles serveur
  - NetExec
---

# Énumération

Avant d'exécuter quoi que ce soit, cartographiez le serveur et votre position en son sein : la version (pour en adapter le comportement), les connexions et bases présentes, et, de façon critique, **ce que le principal courant peut déjà faire**. Une grande partie est lisible par le rôle `public` par défaut.

## Trouver les instances

```bash
# From the network: service discovery and SPN-based discovery in AD
nmap -p1433 --script ms-sql-info <target>
# MSSQL SPNs (MSSQLSvc/...) in the directory reveal every registered instance
setspn -T example.local -Q MSSQLSvc/*
```

## Connexions, bases et rôles

```sql
SELECT @@version;                                  -- version and patch level
SELECT name FROM sys.sql_logins;                   -- SQL logins
SELECT name FROM sys.databases;                    -- databases
SELECT name, type_desc FROM sys.server_principals; -- principals (logins, Windows, roles)
```

## Ce que le principal courant peut faire

```sql
SELECT SUSER_SNAME();                               -- who am I (login)
SELECT IS_SRVROLEMEMBER('sysadmin');                -- 1 if already sysadmin
-- logins I can actually impersonate (tests the IMPERSONATE permission, not mere visibility)
SELECT name FROM sys.server_principals
WHERE type IN ('S','U') AND HAS_PERMS_BY_NAME(name, 'LOGIN', 'IMPERSONATE') = 1;
SELECT * FROM fn_my_permissions(NULL, 'SERVER');    -- my server-level permissions
```

```bash
# NetExec drives the same from the network, including a sysadmin check and query exec
nxc mssql <target> -u user -p pass --local-auth -q "SELECT IS_SRVROLEMEMBER('sysadmin')"
```

## Notes d'exploitation

- La première question est toujours **`IS_SRVROLEMEMBER('sysadmin')`** : si vous êtes déjà sysadmin, allez directement à l'[exécution de commandes](command-execution.md) ; sinon, énumérez les cibles d'[usurpation](impersonation.md) et les [serveurs liés](linked-servers.md).
- `public` peut généralement lire `sys.databases`, `sys.server_principals` et appeler les procédures de coercition, donc l'énumération nécessite rarement des droits particuliers.
- Notez ici les bases **TRUSTWORTHY** et la propriété des bases (`SELECT name, is_trustworthy_on FROM sys.databases`), car elles alimentent les chaînes d'escalade.
- L'accès authentifié Windows compte : une connexion [relayée](coercion-and-relay.md) ou de domaine peut se mapper vers un principal plus privilégié qu'une connexion SQL.

## Outils

- **NetExec `mssql`** (`-q`/`-x`, `--local-auth`) : énumération réseau, vérification sysadmin, exécution de requêtes.
- **Impacket `mssqlclient.py`** : énumération authentifiée interactive.
- **nmap `ms-sql-*`** : découverte non authentifiée d'instances et de versions.

## Références

- [NetExec: MSSQL methodology](https://www.netexec.wiki/mssql-protocol/enumeration)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [Microsoft: sys.server_principals](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-server-principals-transact-sql)
