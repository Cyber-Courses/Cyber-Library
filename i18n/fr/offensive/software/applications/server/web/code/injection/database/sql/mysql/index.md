---
title: "Injection SQL MySQL : techniques, payloads et extraction de données"
description: "Exploiter une injection SQL contre MySQL et MariaDB : syntaxe des commentaires, énumération d'information_schema, et les techniques union, error, blind, time, out-of-band et d'accès aux fichiers."
keywords:
  - injection SQL MySQL
  - injection MariaDB
  - information_schema
  - injection basée sur union
  - injection error-based
  - privilèges de fichier MySQL
---

# MySQL

MySQL (et le fork compatible MariaDB) est la base de données derrière une large part des applications web injectables, donc son dialecte mérite d'être connu précisément. Plusieurs traits façonnent la manière dont l'injection se déroule contre lui.

Les commentaires sont `-- ` (l'espace finale est requise), `#` et `/* */`. Les commentaires versionnés `/*! ... */` exécutent leur contenu uniquement sur MySQL à partir d'un numéro de version embarqué, ce qui fait aussi office d'astuce d'évasion de filtre. Les littéraux de chaîne se concatènent avec `CONCAT()` plutôt que `||` (l'opérateur `||` signifie OU logique sauf si `PIPES_AS_CONCAT` est activé), et les littéraux hexadécimaux tels que `0x7573657273` remplacent les chaînes entre apostrophes quand celles-ci sont filtrées.

Les métadonnées de schéma vivent dans `information_schema` (les bases dans `SCHEMATA`, les tables dans `TABLES`, les colonnes dans `COLUMNS`), qui est l'épine dorsale de l'extraction aveugle et union. `database()`, `user()` et `@@version` identifient le contexte courant. Les requêtes empilées sont généralement indisponibles : les pilotes PHP courants (`mysqli_query`, PDO avec émulation) envoient une instruction par appel, donc les techniques qui dépendent d'une seconde instruction séparée par `;` fonctionnent rarement ici, contrairement à MSSQL ou PostgreSQL.

L'accès aux fichiers est verrouillé. `LOAD_FILE()` et `SELECT ... INTO OUTFILE/DUMPFILE` nécessitent le privilège `FILE`, et la variable système `secure_file_priv` restreint (ou désactive) les répertoires qu'ils peuvent toucher. Ces valeurs par défaut décident si la lecture de fichier, l'écriture de web shell et l'exécution de commandes par UDF sont accessibles.

## Techniques

- **[Enumeration](enumeration.md)** : fingerprinter le moteur, la version, le contexte courant et les privilèges.
- **[Authentication bypass](authentication-bypass.md)** : détourner un login construit à partir des champs de credentials.
- **[Union-based](union-based/index.md)** : ajouter un `UNION SELECT` pour tirer des données dans la réponse visible.
- **[Error-based](error-based/index.md)** : forcer la sortie de la requête dans un message d'erreur reflété.
- **[Boolean blind](blind/index.md)** : déduire les données bit par bit à partir des différences de réponse vrai/faux.
- **[Time-based](time-based/index.md)** : déduire les données à partir de délais de réponse conditionnels.
- **[Out-of-band](out-of-band/index.md)** : exfiltrer via DNS ou SMB quand aucun canal n'est reflété.
- **[File read](read-file.md)** : lire les fichiers du serveur avec `LOAD_FILE()`.
- **[Command execution](command-execution/index.md)** : écrire des web shells ou charger un UDF pour des commandes système.
- **[WAF bypass](waf-bypass/index.md)** : atteindre `information_schema`, `version()` et les mots-clés malgré les filtres.

## Pages

- **[Dump in one shot](dios.md)** : Construire un unique payload union MySQL qui concatène le schéma et les données dans une seule réponse avec des sous-requêtes imbriquées et GROUP_CONCAT.
- **[INSERT-based](insert.md)** : Exploiter une injection à l'intérieur d'une instruction INSERT MySQL : faire fuiter des données via les fonctions d'erreur dans VALUES et écraser des lignes avec ON DUPLICATE KEY UPDATE.
- **[Truncation](truncation.md)** : Abuser de la troncature de chaînes MySQL et du rognage des espaces finales pour entrer en collision avec un compte existant tel qu'admin, quand le mode SQL strict est désactivé.

## Outils

- **sqlmap** : détection et exploitation automatisées de l'injection MySQL à travers les techniques union, error, blind, time et file.
- **ghauri** : alternative rapide avec forte évasion des WAF.
- **mysql** : client officiel pour valider les payloads et exécuter directement les requêtes extraites.

## Références

- MySQL Reference Manual: `information_schema` tables and string functions
- OWASP Testing Guide: Testing for SQL Injection
