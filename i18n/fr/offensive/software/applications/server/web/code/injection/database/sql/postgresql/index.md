---
title: "Injection SQL PostgreSQL : techniques, charges et extraction de données"
description: "Exploiter l'injection SQL contre PostgreSQL : dollar-quoting, casts de chaîne pour les fuites d'erreur, timing pg_sleep, requêtes empilées, pg_read_file et exécution de commandes COPY ... PROGRAM."
keywords:
  - injection SQL PostgreSQL
  - pg_sleep
  - COPY PROGRAM
  - dollar quoting
  - pg_catalog
  - cast basé sur les erreurs
---

# PostgreSQL

PostgreSQL a un dialecte riche qui change plusieurs techniques d'injection par rapport à MySQL, et il tend à récompenser une injection réussie par davantage de puissance car ses primitives fichier et commande sont intégrées.

Les commentaires sont `--` et `/* */`. Les chaînes se concatènent avec l'opérateur standard `||`, et le dollar-quoting (`$$text$$` ou `$tag$text$tag$`) fournit un littéral de chaîne sans guillemets, utile pour contourner les filtres. La gestion des types est stricte : `UNION` exige que les types de colonnes des deux côtés correspondent ou soient explicitement castés, de sorte que les casts `NULL::text` et `value::text` apparaissent partout dans les charges union.

Contrairement aux pilotes MySQL courants, PostgreSQL autorise fréquemment les requêtes empilées : une seconde instruction séparée par `;` s'exécute souvent, ce qui ouvre `CREATE`, `COPY` et le DDL depuis une seule injection. Le catalogue vit à la fois dans le `information_schema` SQL-standard et dans le `pg_catalog` natif (`pg_tables`, `pg_class`, `pg_namespace`, `pg_roles`, `pg_database`), et `pg_catalog` est souvent atteignable quand `information_schema` est filtré.

La puissance dépend du rôle. Un superutilisateur peut lire des fichiers avec `pg_read_file()`, les écrire avec `COPY ... TO` et exécuter des commandes OS avec `COPY ... FROM PROGRAM`. Sur la version 11 et ultérieure, un non-superutilisateur n'atteint chaque primitive que via le rôle prédéfini correspondant : `pg_read_server_files` pour les lectures, `pg_write_server_files` pour les écritures `COPY ... TO`, et `pg_execute_server_program` pour l'exécution de programmes. Vérifier tôt `current_setting('is_superuser')` et ces appartenances de rôle décide des chemins ouverts.

## Techniques

- **[Énumération](enumeration.md)** : prendre l'empreinte de la version, du contexte courant et du rôle.
- **[Contournement d'authentification](authentication-bypass.md)** : subvertir une connexion construite à partir des champs d'identifiants.
- **[Basée sur UNION](union-based/index.md)** : ajouter un `UNION SELECT` avec des casts correspondants.
- **[Basée sur les erreurs](error-based/index.md)** : fuiter des valeurs via les erreurs de cast de type.
- **[Aveugle booléenne](blind/index.md)** : déduire des données à partir de différences de réponse vrai/faux.
- **[Temporelle](time-based.md)** : déduire des données à partir des délais `pg_sleep`.
- **[Requêtes empilées](stacked-query.md)** : exécuter des instructions supplémentaires après un `;`.
- **[Privilèges](privileges.md)** : lire le rôle et ses octrois.
- **[Manipulation de fichiers](file-manipulation.md)** : lire et écrire des fichiers avec `pg_read_file` et `COPY`.
- **[Hors bande](out-of-band.md)** : exfiltrer sur DNS via `COPY ... PROGRAM`.
- **[Exécution de commandes](command-execution/index.md)** : exécuter des commandes OS via `COPY PROGRAM` ou une fonction en langage non de confiance.
- **[Contournement de WAF](waf-bypass.md)** : `CHR()`, dollar-quoting et alternatives de catalogue au-delà des filtres.

## Outils

- **sqlmap** : détection et exploitation automatisées de l'injection PostgreSQL sur toutes les techniques.
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **psql** : client officiel pour valider directement les charges et les requêtes de catalogue.

## Références

- PostgreSQL Documentation: system catalogs, functions, and COPY
- OWASP Testing Guide: Testing for SQL Injection
