---
title: "Injection SQL IBM Db2 : techniques et payloads"
description: "Exploiter l'injection SQL contre IBM Db2 (LUW) : la table à ligne unique SYSIBM.SYSDUMMY1, le catalogue SYSCAT, les registres spéciaux, l'extraction LISTAGG, et l'absence de fonction SLEEP."
keywords:
  - injection SQL Db2
  - SYSIBM.SYSDUMMY1
  - SYSCAT
  - registres spéciaux
  - LISTAGG
  - FETCH FIRST
---

# IBM Db2

IBM Db2 (l'édition LUW sur Linux, Unix et Windows) a un dialecte distinctif, et les payloads de cette page le ciblent plutôt que Db2 for z/OS ou Db2 for i, qui diffèrent.

Comme Oracle, chaque `SELECT` nécessite un `FROM`, et la pseudo-table à ligne unique est `SYSIBM.SYSDUMMY1` (le `DUAL` de Db2). Il n'y a pas de `LIMIT` ; la limitation de lignes se fait avec `FETCH FIRST n ROWS ONLY`. Les chaînes se concatènent avec `||` ou `CONCAT()`, et les commentaires sont `--` et `/* */`. L'identité provient des registres spéciaux `CURRENT USER`, `SESSION_USER`, `SYSTEM_USER`, `CURRENT SCHEMA` et `CURRENT SERVER`.

Le catalogue réside dans les vues `SYSCAT` (`SYSCAT.TABLES` avec `TABSCHEMA`/`TABNAME`, `SYSCAT.COLUMNS` avec `COLNAME`, `SYSCAT.DBAUTH` pour les droits) et l'ancien `SYSIBM.SYSTABLES`. La version et le niveau de service proviennent de `SYSIBMADM.ENV_INST_INFO` et `SYSIBMADM.ENV_SYS_INFO`, pas d'une fonction `version()`. L'agrégation en une seule cellule utilise `LISTAGG` (Db2 9.7 et plus) ou `XMLAGG`.

Deux limites façonnent le choix de technique. Db2 n'a pas de `SLEEP`/`WAITFOR`, donc l'inférence temporelle repose sur une requête délibérément lourde, et ses messages d'erreur renvoient rarement un résultat de requête arbitraire, donc la technique basée sur les erreurs est faible et l'inférence booléenne ou temporelle est le canal aveugle fiable. Les requêtes empilées sont généralement indisponibles via les pilotes standard CLI/JDBC.

## Techniques

- **[Énumération](enumeration.md)** : niveau de service, registres spéciaux et catalogue SYSCAT.
- **[Contournement d'authentification](authentication-bypass.md)** : subvertir un login construit à partir des champs d'identifiants.
- **[Basée sur union](union-based.md)** : ajouter un `UNION SELECT ... FROM SYSIBM.SYSDUMMY1`.
- **[Basée sur les erreurs](error-based.md)** : la surface d'erreur étroite de Db2 et ce qu'elle peut divulguer.
- **[Aveugle](blind.md)** : inférer les données à partir des différences de réponse booléennes.
- **[Temporelle](time-based.md)** : inférer les données avec une requête lourde, puisque Db2 n'a pas de SLEEP.
- **[Privilèges](privileges.md)** : lire les autorités et les droits depuis SYSCAT.DBAUTH.
- **[Exécution de commandes](command-execution.md)** : routines externes, et les limites d'ADMIN_CMD.
- **[DIOS](dios.md)** : dumps en une seule requête avec `XMLAGG`/`LISTAGG`.
- **[Contournement de WAF](waf-bypass.md)** : `CHR()`, concaténation et hexadécimal pour franchir les filtres.

## Outils

- **sqlmap** : détection et exploitation automatisées (`--dbms=Db2`).
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **db2** (ou clpplus) : le client Db2 pour une session directe une fois les identifiants récupérés.

## Références

- IBM Db2 SQL Reference : special registers, catalog views, built-in functions
- OWASP Testing Guide : Testing for SQL Injection
