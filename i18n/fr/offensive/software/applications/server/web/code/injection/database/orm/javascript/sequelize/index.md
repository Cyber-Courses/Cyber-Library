---
order: 1
title: "Injection Sequelize"
description: "Injection via sequelize.query() concaténé, interpolation d'identifiant et objets opérateur contrôlés par l'attaquant."
keywords:
  - Sequelize
  - ORM Node.js
  - requête brute
  - injection d'opérateur
  - replacements
---

# Sequelize

Sequelize propose `replacements` et `bind` pour un paramétrage sûr, mais les chaînes `sequelize.query()` concaténées et les identifiants interpolés tels qu'un `ORDER BY` dynamique réintroduisent l'injection dans les applications Node.js. Un cas connexe, les **objets opérateur** contrôlés par l'attaquant dans une clause `where`, ne s'applique que là où les anciens alias d'opérateur sous forme de chaîne sont activés ou où l'application mappe l'entrée sur les symboles `Op` de Sequelize ; les valeurs par défaut actuelles n'interprètent pas les clés JSON comme `$ne` en tant qu'opérateurs.

## Pages

- **[Injection d'opérateur](operator-injection.md)** : Lorsque du JSON de requête est passé dans une clause where Sequelize, les clés d'opérateur contrôlées par l'attaquant ($gt, $ne, $like) altèrent la logique de requête, contournement d'authentification et d...
- **[Injection de requête brute](raw-query-injection.md)** : sequelize.query() exécute du SQL brut contre le backend ; concaténer des données de requête dedans au lieu d'utiliser replacements ou bind donne une injection SQL dans les applications Node.js...
- **[Injection de replacement](replacement-injection.md)** : Mélanger :replacements avec la concaténation de chaînes, ou alimenter des identifiants contrôlés par l'attaquant via replacements, réintroduit l'injection SQL dans les requêtes brutes Sequelize...

## Outils

- **sqlmap** : exploiter les points d'injection SQL brut sequelize.query() concaténés.
- **Burp Repeater** : tester les identifiants interpolés et les objets opérateur dans les clauses where.

## Références

- Documentation Sequelize : Raw queries
- OWASP : SQL Injection
