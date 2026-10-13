---
order: 2
title: "De l'injection MySQL à l'exécution de commandes"
description: "Les deux voies d'une injection MySQL privilégiée vers l'exécution de code : écrire un web shell avec INTO OUTFILE/DUMPFILE, ou charger un UDF sys_exec."
keywords:
  - exécution de commandes MySQL
  - webshell INTO OUTFILE
  - INTO DUMPFILE
  - lib_mysqludf_sys
  - sys_exec
---

# Exécution de commandes

MySQL n'a aucune fonction intégrée qui exécute une commande système, donc l'exécution de code s'atteint indirectement depuis une injection privilégiée. Il existe deux voies, et toutes deux dépendent du privilège `FILE` et d'un `secure_file_priv` qui autorise les écritures.

La première voie écrit un web shell dans un répertoire servi par le serveur web. `SELECT ... INTO OUTFILE` et `INTO DUMPFILE` écrivent la sortie de la requête dans un fichier sur l'hôte ; viser la racine web y dépose un script ensuite exécuté via HTTP, ce qui convertit l'accès à la base en exécution de commandes au niveau applicatif.

La seconde voie étend le serveur lui-même avec une fonction définie par l'utilisateur. Écrire la bibliothèque partagée `lib_mysqludf_sys` dans le répertoire de plugins et enregistrer `sys_exec`/`sys_eval` fournit une fonction qui exécute des commandes shell directement depuis SQL. Elle nécessite davantage de privilèges (accès en écriture au répertoire de plugins et `CREATE FUNCTION`) mais donne l'exécution de commandes sous le compte de service MySQL sans toucher à la couche web.

## Pages

- **[INTO OUTFILE](into-outfile.md)** : écrire un web shell dans la racine du document.
- **[INTO DUMPFILE](into-dumpfile.md)** : écrire des octets exacts, utilisé pour des binaires tels qu'une bibliothèque UDF.
- **[UDF sys_exec](udf-sys-exec.md)** : charger `lib_mysqludf_sys` pour l'exécution directe de commandes.

## Outils

- **sqlmap** : automatise l'écriture de web shell et l'exécution de commandes par UDF avec `--os-shell`.
- **Metasploit Framework** : module `mysql_udf_payload` pour déployer l'UDF sys_exec.

## Références

- MySQL Reference Manual: SELECT INTO, user-defined functions, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
