---
order: 11
title: "Prise d'empreinte et énumération dans l'injection Oracle"
description: "S'orienter dans une injection Oracle : confirmer le moteur, lire la version et le contexte de session avec SYS_CONTEXT, et lister les privilèges de l'utilisateur courant."
keywords:
  - prise d'empreinte Oracle
  - v$version
  - SYS_CONTEXT
  - session_privs
  - DUAL
---

# Énumération

Confirmez que le moteur est Oracle et lisez le contexte de session avant de choisir une technique. Le besoin de `FROM DUAL` est lui-même une empreinte : un payload qui ne fonctionne qu'avec une clause `FROM` et échoue sans elle pointe vers Oracle.

Lisez la bannière et l'identité. `v$version` renvoie plusieurs lignes, fixez-en donc une avec `ROWNUM` :

```sql
' UNION SELECT banner,NULL,NULL FROM v$version WHERE ROWNUM=1-- 
' UNION SELECT user,SYS_CONTEXT('USERENV','CURRENT_SCHEMA'),SYS_CONTEXT('USERENV','DB_NAME') FROM dual-- 
```

`SYS_CONTEXT('USERENV', ...)` est la source la plus riche de métadonnées de session : `SESSION_USER`, `CURRENT_USER`, `CURRENT_SCHEMA`, `DB_NAME`, `SERVER_HOST`, `OS_USER` et `IP_ADDRESS`. `SELECT user FROM dual` donne le propriétaire du schéma courant, et `global_name` identifie la base de données.

Les privilèges décident quelles primitives sont atteignables, listez donc les rôles et privilèges système de la session :

```sql
' UNION SELECT privilege,NULL,NULL FROM session_privs-- 
' UNION SELECT granted_role,NULL,NULL FROM user_role_privs-- 
```

Voir `DBA`, ou des droits spécifiques tels que `CREATE ANY PROCEDURE`, `CREATE ANY JOB` ou execute sur les paquets `UTL_`, vous indique si l'exécution de commandes, l'accès aux fichiers et les canaux hors bande sont ouverts. Avec la version, le contexte et les privilèges connus, les techniques restantes s'appliquent avec les bonnes attentes.

## Outils

- **sqlmap** : prise d'empreinte et énumération automatisées (`--banner`, `--current-user`, `--privileges`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlplus** (ou SQLcl) : le client Oracle pour lire directement `SYS_CONTEXT` et le dictionnaire de données.

## Références

- Oracle Database SQL Language Reference : SYS_CONTEXT, data dictionary views
- OWASP Testing Guide : Testing for SQL Injection
