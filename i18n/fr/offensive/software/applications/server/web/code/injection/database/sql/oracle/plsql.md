---
order: 8
title: "Injection PL/SQL dans Oracle"
description: "Injecter dans le PL/SQL d'Oracle : blocs anonymes, SQL dynamique construit avec EXECUTE IMMEDIATE, et abus des procédures à droits de définisseur pour exécuter du code en tant que leur propriétaire."
keywords:
  - injection PL/SQL
  - bloc anonyme
  - EXECUTE IMMEDIATE
  - droits de définisseur
  - droits d'invocateur
---

# Injection PL/SQL

Parce qu'Oracle n'autorise pas les requêtes empilées via les pilotes habituels, l'action multi-instructions provient du PL/SQL à la place. Deux situations l'exposent : un bloc anonyme injectable, et une procédure stockée qui construit du SQL dynamique à partir de l'entrée.

Une procédure qui concatène l'entrée dans `EXECUTE IMMEDIATE` est injectable tout comme une requête web. Si le code vulnérable exécute `EXECUTE IMMEDIATE 'SELECT ... WHERE name=''' || p_in || ''''`, un payload ferme le littéral et ajoute de la logique. Le facteur décisif est de savoir avec les droits de qui le code s'exécute.

Une procédure à **droits de définisseur** (la valeur par défaut, `AUTHID DEFINER`) s'exécute avec les privilèges de son propriétaire, pas de l'appelant. Injecter dans une procédure à droits de définisseur détenue par un schéma à privilèges élevés (historiquement plusieurs paquets détenus par `SYS`) exécute le SQL injecté en tant que ce propriétaire, ce qui est la voie classique d'escalade de privilèges Oracle. Une procédure à **droits d'invocateur** (`AUTHID CURRENT_USER`) s'exécute en tant qu'appelant, de sorte qu'elle n'offre aucune escalade mais reste injectable pour l'accès aux données.

Là où le point d'injection est lui-même un bloc PL/SQL, un attaquant peut exécuter un bloc complet :

```sql
'; BEGIN EXECUTE IMMEDIATE 'GRANT DBA TO '||USER; END;-- 
```

À l'intérieur d'un contexte `SYS` à droits de définisseur, ce `GRANT` réussit et fait de l'utilisateur courant un DBA. Le même `EXECUTE IMMEDIATE` exécute du DDL, appelle des paquets et crée des objets, de sorte que l'injection PL/SQL est le pivot d'un bug de lecture de données vers le contrôle total de la base. Trouver quelle procédure atteignable est à droits de définisseur et surprivilégiée est l'étape clé.

## Outils

- **sqlmap** : détecte les paramètres injectables qui atteignent le SQL dynamique.
- **ODAT** (Oracle Database Attacking Tool) : abuse des procédures à droits de définisseur et du PL/SQL pour l'escalade.
- Tests manuels avec Burp Repeater et le client Oracle (sqlplus ou SQLcl).

## Références

- Oracle Database PL/SQL Language Reference : dynamic SQL, invoker's and definer's rights
- OWASP Testing Guide : Testing for SQL Injection
