---
order: 3
title: "Injection ORM Java"
description: "Hibernate et JPA lient les paramètres sur leurs API de requête, mais la construction de chaînes HQL/JPQL, les requêtes natives et la construction dynamique de critères à partir d'une entrée non fiable déjouent ce binding."
keywords:
  - ORM Java
  - injection Hibernate
  - HQL
  - JPQL
  - requête native
---

# Java

La persistance sur JVM passe par Hibernate et le JPA qu'il implémente. Les requêtes sont sûres quand elles sont écrites avec des paramètres liés via `setParameter`, et injectables quand le HQL, le JPQL ou le SQL natif est assemblé par concaténation de chaînes.

## L'ORM ici

- **[Hibernate](hibernate/index.md)** : HQL construit avec la concaténation de `createQuery()`, SQL natif via `createNativeQuery()`, et critères dynamiques construits à partir de l'entrée.

## Le schéma commun

Le HQL et le JPQL opèrent sur des entités mappées et leurs champs plutôt que sur des tables brutes, ce qui façonne les charges utiles mais ne les rend pas sûres : une chaîne `createQuery()` concaténée accepte les mêmes évasions booléennes et par sous-requête que le SQL. Les requêtes natives descendent au SQL brut et portent toute la surface d'injection, y compris `UNION` et les instructions empilées lorsque le pilote le permet. Les critères dynamiques assemblés en sélectionnant des opérateurs ou des champs à partir de l'entrée sont la forme API-ORM de la même faille. Le correctif dans chaque cas est le binding `setParameter` et une forme de requête fixe, donc l'absence de ceux-ci est le signal.

## Outils

- **sqlmap** : exploiter les points d'injection de requête native et de fragment brut d'Hibernate.
- **Burp Repeater et Intruder** : façonner les charges utiles HQL, SQL natif et Criteria.

## Références

- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Hibernate ORM : HQL/JPQL](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql)
