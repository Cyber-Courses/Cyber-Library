---
order: 3
title: "MySQL et MariaDB"
description: "La surface offensive de MySQL et MariaDB atteints en tant que service : s'authentifier, écrire des fichiers de l'hôte via INTO OUTFILE et les lire avec LOAD_FILE, et exécuter des commandes du système d'exploitation via des fonctions définies par l'utilisateur."
keywords:
  - MySQL
  - MariaDB
  - INTO OUTFILE
  - fonction définie par l'utilisateur
  - privilège FILE
---

# MySQL et MariaDB

MySQL et son fork MariaDB (port 3306) offrent deux primitives offensives principales une fois authentifié : **lecture et écriture de fichiers** via `LOAD_FILE` et `INTO OUTFILE` (conditionnées par le privilège `FILE` et `secure_file_priv`), et **exécution de commandes du système d'exploitation** via une fonction définie par l'utilisateur. L'écriture de fichier seule suffit souvent, en déposant un webshell sous un répertoire servi.

## Ce qu'il faut viser

- **[Exécution de commandes](command-execution.md)** : une fonction définie par l'utilisateur (`lib_mysqludf_sys`) qui exécute des commandes OS sous le compte de service.
- **[Accès aux fichiers](file-access.md)** : `INTO OUTFILE` pour écrire un webshell, `LOAD_FILE` pour lire des fichiers de l'hôte.

## Obtenir une session

```bash
# default/weak credentials, commonly root with no or a weak password
mysql -h <target> -u root -p
hydra -L users.txt -P passwords.txt <target> mysql
```

## Références

- [HackTricks: pentesting MySQL (3306)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
- [MySQL: the FILE privilege and secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_file)
