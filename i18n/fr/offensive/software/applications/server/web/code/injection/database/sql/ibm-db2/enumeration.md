---
order: 4
title: "Fingerprinting et énumération dans l'injection IBM Db2"
description: "S'orienter dans une injection IBM Db2 : confirmer le moteur, lire le niveau de service et les registres spéciaux, et cartographier le schéma via le catalogue SYSCAT."
keywords:
  - fingerprinting Db2
  - SYSIBMADM.ENV_INST_INFO
  - registres spéciaux
  - SYSCAT.TABLES
  - SYSIBM.SYSDUMMY1
---

# Énumération

Confirmez que le moteur est Db2 et lisez le contexte avant de choisir une technique. Le besoin de `FROM SYSIBM.SYSDUMMY1` sur les sélections de ligne unique est en soi un fingerprint, tout comme le catalogue `SYSCAT`.

Lisez le niveau de service et l'identité. La version et le fixpack proviennent d'une vue administrative, pas d'une fonction :

```sql
' UNION SELECT SERVICE_LEVEL,FIXPACK_NUM FROM SYSIBMADM.ENV_INST_INFO-- 
' UNION SELECT CURRENT USER,CURRENT SERVER FROM SYSIBM.SYSDUMMY1-- 
```

`SYSIBMADM.ENV_SYS_INFO` ajoute les détails de l'hôte et de l'OS. Les registres spéciaux donnent l'identité : `CURRENT USER` (l'identifiant d'autorisation pour les vérifications), `SESSION_USER` et `SYSTEM_USER` (les identifiants de session et de connexion), `CURRENT SCHEMA`, et `CURRENT SERVER` (la base de données).

Cartographiez le schéma via `SYSCAT`. Listez les tables, puis les colonnes :

```sql
' UNION SELECT TABNAME,TABSCHEMA FROM SYSCAT.TABLES-- 
' UNION SELECT COLNAME,TYPENAME FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

Les noms du catalogue sont stockés en majuscules, faites-les donc correspondre en majuscules. Les anciens `SYSIBM.SYSTABLES` et `SYSIBM.SYSCOLUMNS` sont des alternatives lorsque `SYSCAT` est filtré. Les droits pour l'identifiant d'autorisation courant proviennent de `SYSCAT.DBAUTH` et `SYSCAT.TABAUTH`, que la page sur les privilèges utilise pour décider ce que le reste de l'attaque peut atteindre. Avec le niveau de service, l'identité et le schéma connus, l'extraction se poursuit avec les techniques union, aveugle ou temporelle.

## Outils

- **sqlmap** : fingerprinting et énumération de schéma automatisés (`--banner`, `--tables`, `--columns`).
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **db2** (ou clpplus) : le client Db2 pour lire les registres spéciaux et le catalogue `SYSCAT`.

## Références

- IBM Db2 SQL Reference : special registers, SYSIBMADM administrative views, SYSCAT catalog
- OWASP Testing Guide : Testing for SQL Injection
