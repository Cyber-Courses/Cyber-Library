---
order: 7
title: "Énumérer et élever les privilèges dans une injection MSSQL"
description: "Lire les rôles serveur et base de données du login SQL Server, et les voies d'escalade courantes : usurpation, bases trustworthy et appartenance à sysadmin."
keywords:
  - IS_SRVROLEMEMBER
  - sysadmin
  - EXECUTE AS
  - base trustworthy
  - fn_my_permissions
---

# Privilèges

Les rôles du login déterminent quelles primitives sont accessibles, donc l'énumération des privilèges suit le fingerprinting. Le contrôle principal est l'appartenance aux rôles serveur :

```sql
' UNION SELECT IS_SRVROLEMEMBER('sysadmin'),IS_SRVROLEMEMBER('serveradmin'),NULL-- 
```

`sysadmin` est l'administrateur complet : il peut activer `xp_cmdshell`, lire et écrire des fichiers, extraire les hashs des logins et pivoter via les serveurs liés. Notez que `serveradmin` est un rôle moindre qui gère les paramètres du serveur et n'est pas équivalent à `sysadmin` ; le rôle DBA à viser est `sysadmin`. Les permissions effectives sont listées avec `fn_my_permissions` :

```sql
' UNION SELECT permission_name,NULL,NULL FROM fn_my_permissions(NULL,'SERVER')-- 
```

Quand le login n'est pas `sysadmin`, plusieurs voies d'escalade existent. Les droits d'usurpation (`EXECUTE AS LOGIN = 'sa'`) élèvent la session lorsque le login détient `IMPERSONATE` sur un login privilégié. Une base marquée `TRUSTWORTHY` et possédée par un login à privilèges élevés permet à un module créé en son sein de s'exécuter avec les droits de ce propriétaire, une voie vers `sysadmin` depuis `db_owner`. Les appartenances qui permettent de créer des logins ou d'attribuer des rôles (`ALTER ANY LOGIN`, `CONTROL SERVER`) sont abusables de manière similaire.

Une fois `sysadmin` détenu ou atteint, promouvez un login contrôlé au rang d'administrateur avec :

```sql
'; ALTER SERVER ROLE sysadmin ADD MEMBER [attacker]-- 
```

Connaître le rôle et ses options d'escalade détermine s'il faut viser directement l'exécution de commandes ou enchaîner d'abord une étape d'usurpation ou de base trustworthy.

## Outils

- **sqlmap** : lit les rôles et privilèges du login avec `--privileges`.
- **PowerUpSQL** : `Invoke-SQLAudit` pour énumérer les rôles et les voies d'escalade.

## Références

- Microsoft SQL Server Documentation: IS_SRVROLEMEMBER, fn_my_permissions, EXECUTE AS, TRUSTWORTHY
- OWASP Testing Guide: Testing for SQL Injection
