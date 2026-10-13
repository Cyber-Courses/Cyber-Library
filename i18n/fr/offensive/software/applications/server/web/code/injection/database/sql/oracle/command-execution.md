---
order: 2
title: "Exécution de commandes via une injection Oracle"
description: "Atteindre l'exécution de commandes OS depuis une injection Oracle privilégiée via les procédures stockées Java et les tâches externes DBMS_SCHEDULER."
keywords:
  - exécution de commandes Oracle
  - procédure stockée Java
  - DBMS_SCHEDULER
  - tâche externe
  - DBMS_JAVA
---

# Exécution de commandes

Oracle n'a pas de fonction de commande unique comme le `xp_cmdshell` de SQL Server, de sorte que l'exécution OS provient des runtimes plus riches de la base de données, et chaque voie nécessite un privilège élevé (en pratique DBA ou des droits dangereux spécifiques), généralement atteint d'abord via l'étape d'escalade de privilèges.

La voie Java exécute du code natif. Avec `java.lang.RuntimePermission` accordée, une classe Java chargée peut appeler `Runtime.getRuntime().exec()`, enveloppée dans une fonction PL/SQL et appelée depuis SQL :

```sql
SELECT DBMS_JAVA.RUNJAVA('oracle/aurora/util/Wrapper /bin/sh -c id') FROM dual;
```

ou, plus couramment, en chargeant une petite source Java qui expose une méthode `exec` et en la publiant comme fonction PL/SQL. Cela nécessite les permissions Java et la capacité de créer des sources Java, toutes deux de niveau DBA.

La voie `DBMS_SCHEDULER` exécute directement un programme externe. Une tâche `EXECUTABLE` nécessite à la fois `CREATE JOB` et `CREATE EXTERNAL JOB` (pas l'un ou l'autre seul), ou `CREATE ANY JOB` pour la créer dans un autre schéma. Créez une tâche dont le `job_type` est `EXECUTABLE` :

```sql
BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name=>'x',job_type=>'EXECUTABLE',job_action=>'/bin/sh',number_of_arguments=>2,enabled=>FALSE); ... DBMS_SCHEDULER.ENABLE('x'); END;
```

La tâche s'exécute sous le compte OS configuré pour les tâches externes. Les anciennes versions d'Oracle offraient aussi `DBMS_SCHEDULER` ainsi que les voies héritées de procédure externe (`extproc`) et `PL/SQL` native.

Toutes ces voies nécessitent un contexte PL/SQL injectable (Oracle n'empile pas les instructions SQL simples) et un privilège de niveau DBA, elles constituent donc l'étape finale après l'escalade. Les commandes s'exécutent avec les privilèges OS du serveur Oracle, de sorte que ce compte détermine le point d'ancrage.

## Outils

- **ODAT** (Oracle Database Attacking Tool) : automatise l'exécution de commandes via Java et les tâches externes `DBMS_SCHEDULER`.
- **sqlplus** (ou SQLcl) : le client Oracle pour définir et invoquer la routine externe.
- Tests manuels avec Burp Repeater et le client Oracle.

## Références

- Oracle Database Java Developer's Guide ; PL/SQL Packages and Types Reference : DBMS_SCHEDULER
- OWASP Testing Guide : Testing for SQL Injection
