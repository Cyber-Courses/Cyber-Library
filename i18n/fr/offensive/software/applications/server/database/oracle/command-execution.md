---
title: "Exécution de commandes Oracle : scheduler, Java et tables externes"
description: "Exécuter des commandes du système d'exploitation depuis Oracle Database en tant que compte de service via les jobs externes DBMS_SCHEDULER, les procédures stockées Java et les préprocesseurs de tables externes, les trois points de sortie qu'ODAT automatise une fois que vous détenez un compte suffisamment privilégié."
keywords:
  - DBMS_SCHEDULER
  - Java stored procedure
  - external table
  - ODAT
  - Oracle RCE
---

# Exécution de commandes

Avec un compte privilégié (DBA ou les bons droits `CREATE`/`EXECUTE`), Oracle exécute des commandes du système d'exploitation en tant que son **compte de service** via plusieurs sous-systèmes. `odat` encapsule les trois ; choisissez celui que les privilèges du compte et la version de la base de données permettent.

## Jobs externes DBMS_SCHEDULER

Le scheduler peut exécuter un exécutable externe en tant que job. Il nécessite à la fois `CREATE JOB` **et** `CREATE EXTERNAL JOB` ; le privilège de job externe est ce qui permet effectivement à la commande OS de s'exécuter :

```bash
# ODAT splits the value on whitespace and does not support shell metacharacters (>, |, quotes),
# so point it at a plain executable or a pre-positioned wrapper script, not an inline one-liner
odat dbmsscheduler -s <target> -d <SID> -U user -P pass --exec "/tmp/run.sh"
```

## Procédures stockées Java

Lorsque Java est installé, une procédure stockée encapsulant `Runtime.exec` exécute des commandes dans la JVM de la base de données :

```bash
odat java -s <target> -d <SID> -U user -P pass --exec "id"
```

## Préprocesseur de table externe

Une table externe avec une directive de **préprocesseur** exécute un programme lorsque la table est lue, une voie qui fonctionne sans Java :

```bash
odat externaltable -s <target> -d <SID> -U user -P pass --exec "/path" "id"
# externaltable can also read host files with --getFile (case-sensitive); it has no write option
```

## Notes d'exploitation

- Les commandes s'exécutent en tant que **compte de service Oracle** (`oracle` sous Linux, souvent un compte de service privilégié sous Windows), donc le gain est un accès à l'hôte en tant que ce compte.
- Les trois points de sortie nécessitent des privilèges différents : choisissez en fonction de ce que l'[énumération](access-and-enumeration.md) a montré (`CREATE JOB` **et** `CREATE EXTERNAL JOB` pour le scheduler, `CREATE PROCEDURE` pour Java, `CREATE ANY DIRECTORY` pour les tables externes), et laissez `odat` les essayer tour à tour.
- La voie des **tables externes** lit également les fichiers de l'hôte (`--getFile`), utile lorsque vous avez seulement besoin de voler un fichier plutôt que d'exécuter une commande.
- Une injection PL/SQL dans un paquet à droits `DEFINER` peut fournir le privilège manquant, transformant un compte faible en un compte qui atteint ces points de sortie.

## Outils

- **ODAT** (`dbmsscheduler`, `java`, `externaltable`) : exécution de commandes automatisée sur les trois points de sortie.
- **sqlplus** : exécutez le PL/SQL directement là où vous préférez le contrôle manuel.

## Références

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: Oracle command execution](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: DBMS_SCHEDULER](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_SCHEDULER.html)
