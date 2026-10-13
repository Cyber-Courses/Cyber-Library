---
order: 10
title: "Énumération des autorités et privilèges dans l'injection IBM Db2"
description: "Lire les autorités et les droits de l'identifiant d'autorisation IBM Db2 courant depuis SYSCAT.DBAUTH pour décider quelles primitives, y compris la création de routines, sont atteignables."
keywords:
  - privilèges Db2
  - SYSCAT.DBAUTH
  - DBADM
  - SYSADM
  - identifiant d'autorisation
---

# Privilèges

Les autorités détenues par l'identifiant d'autorisation courant déterminent jusqu'où va une injection, donc l'énumération des privilèges suit le fingerprinting. Les autorités au niveau base de données se trouvent dans `SYSCAT.DBAUTH`. Une autorité peut être accordée directement à l'utilisateur, à un groupe ou un rôle auquel il appartient, ou à `PUBLIC`, donc filtrer sur le seul utilisateur manque les droits hérités ; incluez ces bénéficiaires :

```sql
' UNION SELECT GRANTEE,GRANTEETYPE,DBADMAUTH FROM SYSCAT.DBAUTH WHERE GRANTEE=CURRENT USER OR GRANTEE='PUBLIC' OR GRANTEETYPE='R'-- 
```

La routine d'autorité effective `SYSPROC.AUTH_LIST_AUTHORITIES_FOR_AUTHID(CURRENT USER,'U')` résout l'ensemble complet, y compris l'héritage de rôle et de groupe, ce qui est la vérification fiable. `DBADMAUTH='Y'` signifie que l'identifiant détient `DBADM` (administration de base de données), qui est large. Les autorités au niveau instance `SYSADM`, `SYSCTRL` et `SYSMAINT` proviennent de la configuration du gestionnaire de base de données plutôt que du catalogue, et `SECADM` régit les objets de sécurité. Les droits d'objet sont dans `SYSCAT.TABAUTH` (privilèges de table) et `SYSCAT.ROUTINEAUTH` (execute sur les routines) :

```sql
' UNION SELECT TABSCHEMA,TABNAME,SELECTAUTH FROM SYSCAT.TABAUTH WHERE GRANTEE=CURRENT USER OR GRANTEE='PUBLIC' OR GRANTEETYPE='R'-- 
```

Les autorités qui comptent pour l'escalade sont celles qui autorisent la création et l'exécution de routines. `CREATE_EXTERNAL_ROUTINE` (et `DBADM`) permettent à l'identifiant de créer des routines externes C ou Java, ce qui est la voie Db2 vers l'exécution de commandes ; `IMPLICIT_SCHEMA` et des droits `GRANT` larges aident à planter des objets. Un identifiant avec `DBADM` ou `SYSADM` peut aussi s'accorder davantage d'autorités.

Connaître les autorités indique s'il faut poursuivre l'exécution de commandes via une routine externe, rester sur l'extraction de données, ou d'abord escalader en s'accordant les autorités manquantes là où l'identifiant courant y est autorisé. Le registre spécial `CURRENT USER` identifie à qui appartiennent ces droits.

## Outils

- **sqlmap** : énumère les privilèges de l'utilisateur courant (`--privileges`) contre Db2.
- **db2** (ou clpplus) : le client Db2 pour lire `SYSCAT.DBAUTH` et `AUTH_LIST_AUTHORITIES_FOR_AUTHID`.
- Test manuel avec Burp Repeater et le client Db2.

## Références

- IBM Db2 SQL Reference and Administration Guide : authorities, SYSCAT.DBAUTH, SYSCAT.ROUTINEAUTH
- OWASP Testing Guide : Testing for SQL Injection
