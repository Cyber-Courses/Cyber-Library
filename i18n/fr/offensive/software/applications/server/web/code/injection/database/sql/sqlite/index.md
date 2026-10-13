---
title: "Injection SQL SQLite : techniques et payloads"
description: "Exploiter l'injection SQL contre SQLite : énumération via sqlite_master, typage faible, vraies primitives d'erreur, temporisation par randomblob, écritures de fichiers par ATTACH et RCE par load_extension."
keywords:
  - injection SQL SQLite
  - sqlite_master
  - load_extension
  - ATTACH DATABASE
  - randomblob
  - base de données embarquée
---

# SQLite

SQLite est une base de données embarquée basée sur fichier, sans serveur ni comptes utilisateur, donc l'injection contre elle a une forme différente de celle des moteurs client-serveur. Plusieurs traits comptent.

Il n'y a pas d'`information_schema` ; le schéma réside dans `sqlite_master` (aliasé `sqlite_schema` dans les versions récentes), dont la colonne `sql` stocke l'instruction `CREATE` de chaque objet et révèle donc directement les noms de colonnes. Il n'y a pas de `SLEEP`, donc l'inférence time-based utilise une expression délibérément coûteuse telle que `randomblob()`. SQLite est faiblement typé, ce qui rend `UNION` facile (les incompatibilités de type la bloquent rarement) mais casse certaines astuces d'erreur des autres moteurs : la division par zéro renvoie `NULL` plutôt qu'une erreur, et caster une chaîne non numérique produit `0`, donc l'injection error-based doit utiliser des primitives qui lèvent réellement une erreur.

Les commentaires sont `--` et `/* */`. La concaténation de chaînes est `||`. Les requêtes empilées dépendent de l'API : `sqlite3_exec` exécute plusieurs instructions séparées par `;`, tandis qu'une instruction préparée n'exécute que la première, donc l'empilement se confirme, il ne se suppose pas.

Les primitives à fort impact sont `ATTACH DATABASE`, qui peut écrire un fichier, et `load_extension()`, qui charge une bibliothèque partagée pour l'exécution de code mais est désactivée par défaut. Il n'y a pas de niveaux de privilège à l'intérieur de SQLite : tout ce que le handle de base de données de l'application peut faire, l'injection peut le faire.

## Techniques

- **[Énumération](enumeration.md)** : version et schéma depuis `sqlite_master`.
- **[Contournement d'authentification](authentication-bypass.md)** : subvertir un login construit à partir des champs d'identifiants.
- **[Union-based](union-based.md)** : ajouter un `UNION SELECT`, facilité par le typage faible.
- **[Error-based](error-based.md)** : faire fuiter des valeurs via des primitives qui lèvent réellement des erreurs.
- **[Blind](blind.md)** : inférer des données à partir des différences de réponse booléennes.
- **[Time-based](time-based.md)** : inférer des données avec des expressions lourdes `randomblob()`.
- **[Manipulation de fichiers](file-manipulation.md)** : écrire des fichiers avec `ATTACH` et `VACUUM INTO`.
- **[Exécution de code à distance](remote-code-execution.md)** : `load_extension()` et le web shell ATTACH.
- **[Techniques d'évasion](evasion-techniques.md)** : commentaires, hex et encodage pour passer les filtres.

## Outils

- **sqlmap** : détection et exploitation automatisées (`--dbms=SQLite`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlite3** : la CLI du moteur pour reproduire le comportement de `sqlite_master`, `ATTACH` et `load_extension`.

## Références

- Documentation SQLite : `sqlite_master`, fonctions de base, ATTACH, load_extension
- OWASP Testing Guide: Testing for SQL Injection
