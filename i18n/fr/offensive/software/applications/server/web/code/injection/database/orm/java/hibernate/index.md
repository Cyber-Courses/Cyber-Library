---
title: "Injection Hibernate"
description: "Injection via HQL/JPQL concaténé, requêtes SQL natives et fragments bruts de l'API Criteria."
keywords:
  - Hibernate
  - HQL
  - JPQL
  - requête native
  - API Criteria
---

# Hibernate

Hibernate prend en charge les paramètres liés, mais le HQL/JPQL assemblé par concaténation de chaînes, les requêtes SQL natives (`createNativeQuery`) et les fragments bruts `Restrictions.sqlRestriction` réintroduisent l'injection SQL contre le backend JDBC sous-jacent. Les noms de propriété ou de tri Criteria contrôlés par l'attaquant sont un problème connexe mais distinct : Hibernate les résout d'après les propriétés mappées plutôt que de les insérer dans le SQL, donc ils peuvent exposer des champs ou un ordonnancement imprévus sans être une injection SQL.

## Pages

- **[Injection de l'API Criteria](criteria-api-injection.md)** : L'API Criteria est sûre côté paramètre pour les valeurs, mais Restrictions.sqlRestriction() incorpore du SQL brut et les noms de propriété contrôlés par l'utilisateur atteignent des colonnes imprévues.
- **[Injection HQL](hql-injection.md)** : Concaténer l'entrée utilisateur dans le HQL/JPQL de createQuery() contourne le binding de paramètres d'Hibernate, exposant les données d'entités via la logique booléenne et des sous-requê...
- **[Injection SQL native](native-sql-injection.md)** : Les API de requête native d'Hibernate exécutent du SQL brut du backend ; concaténer l'entrée utilisateur dans celles-ci donne une injection SQL complète, y compris les requêtes empilées et les fonctions spécifiques au SGBD...

## Outils

- **sqlmap** : automatiser l'exploitation du point d'injection SQL brut createNativeQuery.
- **Burp Repeater et Intruder** : délivrer des charges utiles HQL, SQL natif et Criteria.

## Références

- Hibernate ORM User Guide : HQL and native SQL queries
- OWASP : SQL Injection
