---
order: 12
title: "Fingerprinting et énumération dans l'injection MSSQL"
description: "S'orienter dans une injection MSSQL : confirmer le moteur, lire la version et le contexte courant, et vérifier si le login est sysadmin."
keywords:
  - fingerprinting MSSQL
  - SERVERPROPERTY
  - SYSTEM_USER
  - IS_SRVROLEMEMBER
  - détection de version
---

# Énumération

Confirmez que le moteur est SQL Server et lisez le contexte du login avant de choisir une technique, car l'exécution de commandes, l'accès aux fichiers et les pivots vers les serveurs liés dépendent tous du rôle serveur du login.

`@@version` et `SERVERPROPERTY()` fingerprint le serveur, et les fonctions d'identité donnent le login courant et la base de données :

```sql
' UNION SELECT @@version,SYSTEM_USER,DB_NAME()-- 
```

`SYSTEM_USER` est le login, `USER_NAME()` l'utilisateur de base de données auquel il est mappé, `DB_NAME()` la base de données courante, et `@@SERVERNAME`/`HOST_NAME()` identifient l'instance et l'hôte client. Pour une logique de version précise, `SERVERPROPERTY('ProductVersion')`, `SERVERPROPERTY('ProductLevel')` et `SERVERPROPERTY('Edition')` sont plus propres que l'analyse de la bannière `@@version`.

La vérification décisive est l'appartenance au rôle serveur, puisque `sysadmin` débloque `xp_cmdshell`, l'accès aux fichiers et l'abus des serveurs liés :

```sql
' UNION SELECT IS_SRVROLEMEMBER('sysadmin'),IS_SRVROLEMEMBER('serveradmin'),CURRENT_USER-- 
```

`IS_SRVROLEMEMBER('sysadmin')` retourne `1` quand le login est administrateur complet, ce qui est le fait le plus utile pour planifier le reste de l'attaque. Un non-sysadmin peut tout de même être escaladé dans certaines configurations (une base de données trustworthy avec un propriétaire élevé, ou l'impersonation via `EXECUTE AS`), ce que couvre la page sur les privilèges. Avec la version et le rôle connus, les techniques restantes s'appliquent avec les bonnes attentes.

## Outils

- **sqlmap** : fingerprint le moteur et lit la version, le login et le rôle serveur.
- **PowerUpSQL** : `Get-SQLServerInfo` pour l'énumération de l'instance et du rôle.
- **sqlcmd** : client officiel pour confirmer directement `@@version` et `IS_SRVROLEMEMBER`.

## Références

- Microsoft SQL Server Documentation : SERVERPROPERTY, IS_SRVROLEMEMBER, system functions
- OWASP Testing Guide : Testing for SQL Injection
