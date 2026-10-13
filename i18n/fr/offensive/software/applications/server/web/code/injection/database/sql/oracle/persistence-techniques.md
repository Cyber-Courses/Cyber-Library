---
order: 9
title: "Techniques de persistance dans Oracle après injection"
description: "Maintenir l'accès à une base de données Oracle après une injection privilégiée via les tâches du planificateur, les triggers et des procédures et comptes de porte dérobée."
keywords:
  - persistance Oracle
  - tâche DBMS_SCHEDULER
  - porte dérobée par trigger
  - porte dérobée par procédure stockée
  - persistance via les droits
---

# Persistance

Une fois qu'une injection a atteint un contrôle de niveau DBA, la persistance maintient cet accès à travers les changements de mot de passe et les correctifs. Toutes ces techniques nécessitent le privilège élevé obtenu via l'escalade, et s'exécutent depuis un contexte PL/SQL injectable.

Une tâche du planificateur rétablit l'accès sur une minuterie. Une tâche `DBMS_SCHEDULER` qui réattribue un rôle ou exécute un programme externe se déclenche selon sa planification, indépendamment des réinitialisations d'identifiants ultérieures :

```sql
BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name=>'sync$',job_type=>'PLSQL_BLOCK',job_action=>'BEGIN EXECUTE IMMEDIATE ''GRANT DBA TO lowpriv''; END;',repeat_interval=>'FREQ=HOURLY',enabled=>TRUE); END;
```

Un trigger plante du code qui s'exécute sur un évènement. Un trigger de connexion ou de DML détenu par un schéma privilégié, ou un trigger qui réajoute un privilège supprimé, survit jusqu'à ce qu'il soit remarqué. Une procédure stockée de porte dérobée avec des droits de définisseur dans un schéma privilégié offre une escalade à la demande : l'appeler réattribue DBA ou exécute une commande.

Une option plus discrète est un compte ou un droit dormant : créer un utilisateur discret avec une attribution de rôle, ou ajouter le schéma de l'attaquant à un rôle rarement audité, laisse un identifiant pour revenir. Nommer les objets pour qu'ils se fondent avec ceux d'Oracle (noms de style `SYS`, suffixes `$`) retarde la découverte.

Chacune de ces techniques est une action post-compromission qui suppose déjà DBA, elle fait donc suite à l'escalade de privilèges et à l'exécution de commandes plutôt que de tenir seule. La valeur réside dans la durabilité : une seule tâche ou un seul trigger recrée l'accès qu'un intervenant en réponse à incident supprime ailleurs.

## Outils

- **ODAT** (Oracle Database Attacking Tool) : planifie des tâches et exécute du code sur une instance compromise.
- **sqlplus** (ou SQLcl) : le client Oracle pour installer des tâches du planificateur, des triggers et des procédures de porte dérobée.
- Tests manuels avec Burp Repeater et le client Oracle.

## Références

- Oracle Database PL/SQL Packages and Types Reference : DBMS_SCHEDULER ; SQL Language Reference : CREATE TRIGGER
- OWASP Testing Guide : Testing for SQL Injection
