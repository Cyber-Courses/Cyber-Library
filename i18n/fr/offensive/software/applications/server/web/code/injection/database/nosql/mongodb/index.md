---
order: 3
title: "Injection MongoDB"
description: "Injection d'opérateur et de syntaxe contre MongoDB, où le JSON de la requête atteint une requête sans assainissement et où des objets contrôlés par l'attaquant sont interprétés comme des opérateurs de requête."
keywords:
  - injection MongoDB
  - injection NoSQL
  - injection d'opérateur
  - opérateur de requête
  - type juggling
  - contournement d'authentification
---

# MongoDB

MongoDB stocke des documents et est interrogé avec des objets de requête BSON/JSON plutôt qu'avec un langage de requête textuel, de sorte que la primitive d'injection classique n'est pas une évasion de guillemets mais un **opérateur**. Un filtre MongoDB est une structure comme `{ "username": "admin", "password": "secret" }`, où une valeur nue signifie une égalité. Lorsque la valeur d'une clé est elle-même un objet tel que `{ "$ne": null }`, cet objet est un opérateur de requête (`$ne`, `$gt`, `$regex`, `$where`, et d'autres). L'injection survient lorsque des données contrôlées par l'attaquant atteignent le filtre en tant que structure au lieu d'un scalaire converti : un champ que l'application attend sous forme de chaîne arrive comme objet opérateur, et la comparaison que le développeur avait prévue est remplacée par une que l'attaquant a choisie.

La précondition décisive est que l'application passe des objets contrôlés par l'attaquant dans la requête sans les convertir en chaînes. Cela atteint la requête lorsque le JSON de la requête est étalé directement dans un filtre, ou lorsqu'un analyseur de query-string ou de corps de formulaire (qs, body-parser, PHP) transforme des clés entre crochets comme `password[$ne]=` en objets imbriqués. Ce sous-arbre couvre le contournement d'authentification, le type-juggling JSON et les opérateurs de requête individuels détournés pour le contournement, l'extraction aveugle et le déni de service.

## Sous-thèmes

- **[Opérateur](operator/index.md)** : lorsque des objets contrôlés par l'attaquant atteignent un filtre MongoDB, les opérateurs de requête remplacent la comparaison prévue, permettant le contournement, l'extraction aveugle et le déni de service...

## Pages

- **[Contournement d'authentification](authentication-bypass.md)** : remplacer une chaîne de mot de passe par un objet opérateur tel que {\"$ne\":null} transforme une recherche de connexion en filtre toujours vrai, permettant la connexion sans identifiants.
- **[Injection JSON](json.md)** : une valeur attendue sous forme de chaîne arrivant comme objet ou tableau transforme les comparaisons MongoDB en injection d'opérateur ; les clés dupliquées et l'imbrication construite par l'analyseur élargissent...

## Outils

- **NoSQLMap** : injection d'opérateur et énumération MongoDB automatisées.
- **nosqli** : scanner d'injection d'opérateur MongoDB dans les paramètres web.
- **mongosh** : le shell MongoDB pour valider les filtres contre une base de données cible.
- **Burp Suite** : intercepter et altérer les requêtes HTTP adossées à une requête de base de données.

## Références

- [MongoDB : Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
