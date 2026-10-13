---
order: 10
title: "Élévation de privilèges dans l'injection Oracle"
description: "Passer d'un schéma Oracle peu privilégié à DBA via l'injection dans un package à droits du définisseur, des privilèges système dangereux et l'abus de rôles."
keywords:
  - élévation de privilèges Oracle
  - GRANT DBA
  - CREATE ANY PROCEDURE
  - droits du définisseur
  - CREATE ANY JOB
---

# Élévation de privilèges

Un compte Oracle peu privilégié arrête rarement un attaquant, car Oracle fournit de nombreux chemins d'un schéma ordinaire vers DBA. L'objectif est presque toujours d'exécuter `GRANT DBA TO <me>` (ou `ALTER USER`) dans un contexte autorisé à le faire.

Le chemin classique consiste à injecter dans une procédure à droits du définisseur appartenant à un schéma privilégié. Un package vulnérable appartenant à `SYS` qui construit du SQL dynamique à partir de l'entrée exécute l'instruction injectée en tant que `SYS`, si bien qu'un payload `GRANT DBA TO SCOTT` réussit. Au fil des années, une longue liste de packages fournis se sont révélés exploitables de cette manière, ce qui explique pourquoi l'application des correctifs et le moindre privilège sur le PL/SQL comptent.

Les privilèges système dangereux détenus directement constituent l'autre chemin. `CREATE ANY PROCEDURE` permet de créer une procédure à droits du définisseur dans un schéma privilégié, mais la créer ne suffit pas à elle seule : il faut aussi un moyen de l'invoquer en tant que ce propriétaire, via `EXECUTE ANY PROCEDURE`, un octroi d'exécution explicite ou un gadget qui l'exécute (une tâche du planificateur ou un déclencheur que le propriétaire active). `CREATE ANY TRIGGER` permet d'implanter un déclencheur qui s'exécute en tant que propriétaire privilégié, ce qui est en soi un tel gadget d'invocation. `CREATE ANY JOB` ou `CREATE EXTERNAL JOB` atteint l'exécution de commandes OS via `DBMS_SCHEDULER`. `EXECUTE ANY PROCEDURE` combiné à un package vulnérable aboutit à une élévation. Énumérez d'abord ce que détient la session :

```sql
' UNION SELECT privilege,NULL,NULL FROM session_privs-- 
```

Les permissions Java constituent un chemin supplémentaire : un schéma qui s'est vu accorder `java.io.FilePermission` ou `java.lang.RuntimePermission` peut charger et exécuter du code avec les privilèges OS du serveur de base de données. Le schéma récurrent consiste à trouver un privilège surdimensionné ou une procédure privilégiée injectable, puis à l'utiliser pour s'octroyer `DBA`, après quoi l'exécution de commandes, l'accès aux fichiers et la persistance s'ouvrent tous.

## Outils

- **ODAT** (Oracle Database Attacking Tool) : teste et abuse des privilèges dangereux et des packages à droits du définisseur.
- **sqlplus** (ou SQLcl) : le client Oracle pour exécuter `GRANT` et lire `session_privs`.
- Test manuel avec Burp Repeater et le client Oracle.

## Références

- Oracle Database Security Guide : system privileges, definer's rights
- OWASP Testing Guide : Testing for SQL Injection
