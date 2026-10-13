---
title: "Injection SQL dans Oracle Database : techniques et payloads"
description: "Exploiter l'injection SQL contre Oracle Database : le FROM DUAL obligatoire, les paquets UTL conditionnés par des ACL, le timing DBMS_PIPE, les blocs PL/SQL et les vues de catalogue ALL_."
keywords:
  - injection SQL Oracle
  - DUAL
  - UTL_HTTP UTL_INADDR
  - DBMS_PIPE
  - all_tables
  - injection PL/SQL
---

# Oracle

Oracle Database a le dialecte le plus distinctif des grands moteurs, et plusieurs de ses particularités décident de la façon dont les payloads d'injection sont écrits.

Chaque `SELECT` a besoin d'un `FROM`, de sorte que les requêtes à une seule ligne sélectionnent `FROM DUAL`. Il n'y a pas de `LIMIT` ; la limitation de lignes est `WHERE ROWNUM=1` ou `FETCH FIRST n ROWS ONLY` (12c et au-delà). Les chaînes se concatènent avec `||`, et `CHR()` les construit sans guillemets. Les commentaires sont `--` et `/* */`. Point crucial, Oracle ne prend pas en charge les requêtes empilées via les pilotes JDBC et OCI habituels : une seconde instruction séparée par `;` ne s'exécute pas, de sorte que les actions multi-instructions nécessitent un bloc PL/SQL injectable à la place.

Le catalogue est constitué des vues `ALL_`/`USER_`/`DBA_` (`all_tables`, `all_tab_columns`, `all_users`) et des vues de performance `V$` (`v$version`, `v$instance`). L'identité et le contexte proviennent de `SELECT user FROM dual` et de `SYS_CONTEXT('USERENV', ...)`.

Les primitives les plus puissantes d'Oracle vivent dans des paquets PL/SQL fournis, et depuis la 11g leur portée réseau (`UTL_HTTP`, `UTL_INADDR`, `UTL_TCP`, `UTL_SMTP`, `HTTPURITYPE`) est conditionnée par des listes de contrôle d'accès (ACL) fines, de sorte que le hors bande et certains canaux d'erreur dépendent d'une attribution d'ACL en plus du privilège execute sur le paquet. Indiquez ces conditions préalables quand un payload en dépend.

## Techniques

- **[Énumération](enumeration.md)** : version, contexte de session et utilisateur courant.
- **[Contournement d'authentification](authentication-bypass.md)** : subvertir une connexion construite à partir des champs d'identifiants.
- **[Basée sur UNION](union-based.md)** : ajouter un `UNION SELECT ... FROM DUAL` avec des types correspondants.
- **[Basée sur les erreurs](error-based.md)** : divulguer des valeurs via des erreurs ORA qui renvoient l'entrée.
- **[Aveugle](blind.md)** : déduire des données à partir des différences de réponse booléennes.
- **[Basée sur le temps](time-based.md)** : déduire des données avec `DBMS_PIPE.RECEIVE_MESSAGE`.
- **[Escalade de privilèges](privilege-escalation.md)** : atteindre DBA via l'abus de rôles et de paquets.
- **[Injection PL/SQL](plsql.md)** : injecter dans les blocs anonymes et le SQL dynamique.
- **[Manipulation de fichiers](file-manipulation.md)** : lire et écrire des fichiers avec `UTL_FILE` et des objets directory.
- **[Hors bande](out-of-band.md)** : exfiltrer via DNS et HTTP avec les paquets `UTL`.
- **[Abus de XML et XXE](xml-and-xxe-abuse.md)** : abuser de l'analyse `XMLType` pour le SSRF et la lecture de fichiers.
- **[Exécution de commandes](command-execution.md)** : exécuter des commandes OS via Java ou `DBMS_SCHEDULER`.
- **[Énumération du listener et du SID](listener-and-sid-enumeration.md)** : identifier l'instance et le SID.
- **[Persistance](persistence-techniques.md)** : tâches du planificateur, triggers et portes dérobées.
- **[Techniques d'évasion](evasion-techniques.md)** : commentaires, `CHR()` et astuces de casse pour passer les filtres.

## Outils

- **sqlmap** : détection et exploitation automatisées (`--dbms=Oracle`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **ODAT** (Oracle Database Attacking Tool) : post-exploitation (abus de fichiers, de commandes et de privilèges).
- **sqlplus** (ou SQLcl) : le client Oracle pour une session directe une fois les identifiants récupérés.

## Références

- Oracle Database SQL Language Reference and PL/SQL Packages and Types Reference
- OWASP Testing Guide : Testing for SQL Injection
