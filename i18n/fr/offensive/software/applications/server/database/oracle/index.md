---
title: "Oracle Database"
order: 4
description: "La surface offensive d'Oracle Database atteinte via le listener TNS : énumérer les SID et les services, tester les comptes par défaut et faibles, et exécuter des commandes du système d'exploitation via Java, le scheduler et les tables externes."
keywords:
  - Oracle
  - TNS listener
  - SID
  - ODAT
  - default credentials
---

# Oracle

Oracle Database est atteint via le **listener TNS** (port 1521), qui route un client vers une instance de base de données nommée par un **SID** ou un nom de service. La voie d'attaque est échelonnée : énumérer le listener et les SID valides, tester les comptes (Oracle est tristement célèbre pour ses **identifiants par défaut**), puis, avec un compte de base de données, atteindre l'exécution de commandes du système d'exploitation via Java, le scheduler ou les tables externes. `odat` en automatise l'essentiel.

## Vers quoi se tourner

- **[Accès et énumération](access-and-enumeration.md)** : énumération TNS et SID, comptes par défaut et faibles, extraction du schéma et des empreintes.
- **[Exécution de commandes](command-execution.md)** : commandes OS via `DBMS_SCHEDULER`, procédures stockées Java et tables externes.
- **[Accès aux fichiers](file-access.md)** : lire et écrire des fichiers de l'hôte via `UTL_FILE` et un objet directory.
- **[Hors bande](out-of-band.md)** : SSRF, capture NetNTLM sous Windows et exfiltration en aveugle via les paquets sortants.

## Références

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: pentesting Oracle TNS (1521)](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
