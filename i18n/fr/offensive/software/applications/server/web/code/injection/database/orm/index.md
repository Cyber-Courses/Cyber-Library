---
order: 2
title: "Injection ORM"
description: "Les mappeurs objet-relationnel paramètrent les requêtes normales, mais chacun expose des échappatoires qui rouvrent l'injection, organisées par framework."
keywords:
  - injection ORM
  - mapping objet-relationnel
  - SQL brut
  - query builder
  - injection de second ordre
---

# ORM

Les mappeurs objet-relationnel paramètrent les requêtes ordinaires, mais chaque ORM expose des **échappatoires**, méthodes SQL brutes, fragments de requête construits par chaîne, et identifiants ou opérateurs contrôlés par l'utilisateur, qui réintroduisent l'injection. Les API vulnérables sont propres à chaque ORM, et les ORM sont propres à un langage, donc ce sous-arbre est organisé **par langage, puis par ORM** : le point d'injection dans Django n'est pas celui dans Hibernate ou Sequelize, et un nouvel ORM se place sous le langage auquel il appartient plutôt que dans une liste à plat.

## Organisé par langage

- **[JavaScript](javascript/index.md)** : Sequelize, Prisma, Drizzle et autres query builders Node et TypeScript, où l'échappatoire de requête brute et les objets opérateur ou filtre contrôlés par l'attaquant sont les points d'injection.
- **[Python](python/index.md)** : l'ORM Django et SQLAlchemy, où `raw`/`extra`, `text()` et la construction de lookup ou de filtre à partir de l'entrée contournent le binding.
- **[Java](java/index.md)** : Hibernate et JPA, où la construction de chaînes HQL/JPQL, les requêtes natives et les critères dynamiques déjouent les paramètres.
- **[C#](c-sharp/index.md)** : Entity Framework, où les API SQL brutes et l'interpolation de chaîne non sûre atteignent la base de données sans paramétrage.

Dans tous les cas, le schéma est le même : l'ORM est sûr sur son chemin typé et paramétré, et l'injection vit dans l'API vers laquelle un développeur se tourne quand ce chemin n'est pas pratique.

## Outils

- **sqlmap** : exploiter les échappatoires SQL brutes exposées par n'importe quel ORM.
- **Burp Repeater et Intruder** : façonner les charges utiles pour les points d'injection ORM et les objets opérateur ou filtre.

## Références

- OWASP : SQL Injection
- OWASP Web Security Testing Guide : Testing for ORM Injection
