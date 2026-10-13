---
order: 10
title: "Injection SQL basée sur UNION dans PostgreSQL"
description: "Ajouter UNION SELECT dans PostgreSQL pour extraire des données, en gérant sa correspondance stricte des types avec des casts explicites et en lisant les catalogues pg_catalog et information_schema."
keywords:
  - injection SQL basée sur union
  - UNION SELECT PostgreSQL
  - cast de type
  - pg_catalog
  - string_agg
---

# Basée sur UNION

Un `UNION SELECT` ajoute des lignes choisies par l'attaquant à un résultat renvoyé. PostgreSQL ajoute une subtilité par rapport à MySQL : son système de types est strict, de sorte que chaque colonne du `SELECT` ajouté doit correspondre au type de la colonne d'origine ou être castée explicitement. Caster tout en `text` avec `::text` (ou sélectionner `NULL`, qui convient à tout type) contourne les incompatibilités de type.

Le déroulé est le même qu'ailleurs : détecter le nombre de colonnes, trouver une colonne de texte reflétée, puis lire le catalogue et les données à travers elle. PostgreSQL expose l'information de schéma à la fois dans le `information_schema` SQL-standard et dans le `pg_catalog` natif, et `string_agg()` est son équivalent de `GROUP_CONCAT` pour regrouper de nombreuses lignes dans une seule cellule.

```sql
' UNION SELECT NULL,string_agg(table_name,','),NULL FROM information_schema.tables WHERE table_schema='public'-- 
```

## Pages

- **[Détecter le nombre de colonnes](detect-column-count.md)** : sondage `ORDER BY` et `UNION SELECT NULL`, avec des casts.
- **[Extraire le schéma et les données](extract-schema.md)** : énumérer et dumper via `information_schema` et `pg_catalog`.

## Outils

- **sqlmap** : détection et extraction automatisées basées sur UNION.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: UNION, type casts, `string_agg`
- OWASP Testing Guide: Testing for SQL Injection
