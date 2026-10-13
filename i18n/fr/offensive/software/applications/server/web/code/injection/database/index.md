---
order: 3
title: "Injection de base de données"
description: "Une entrée non fiable qui modifie la requête qu'un serveur envoie à son magasin de données, SQL, NoSQL, portes de sortie des ORM et services d'annuaire."
keywords:
  - injection de base de données
  - injection SQL
  - injection NoSQL
  - injection ORM
  - injection LDAP
---

# Base de données

L'injection de base de données survient lorsqu'une entrée non fiable modifie la requête qu'un serveur envoie à son magasin de données. Elle couvre le **SQL** classique (requêtes construites par chaînes de caractères, tous moteurs confondus), le **NoSQL** (injection d'opérateurs et de syntaxe dans les magasins document, clé-valeur, wide-column et graphe), les portes de sortie des **ORM** qui réintroduisent des requêtes brutes, et les services d'annuaire comme **LDAP**. L'impact va du contournement d'authentification et de la divulgation de données jusqu'à, selon le moteur et les privilèges, l'accès aux fichiers et l'exécution de commandes.

## Sous-thèmes

- **[LDAP](ldap/index.md)** : Exploitation de l'injection LDAP lorsque le code applicatif construit des filtres d'annuaire, des noms distinctifs ou des bases de recherche à partir d'une entrée non fiable, y compris l'authentification...
- **[NoSQL](nosql/index.md)** : Injection contre les magasins de données non relationnels, où des objets, opérateurs ou fragments de langage de requête contrôlés par l'attaquant sont interprétés par le pilote plutôt...
- **[ORM](orm/index.md)** : Les mappeurs objet-relationnel paramètrent les requêtes normales, mais chacun expose des portes de sortie qui rouvrent l'injection, organisées par framework.
- **[SQL](sql/index.md)** : L'injection SQL organisée par moteur de base de données, car la syntaxe exploitable, les fonctions, le catalogue et les primitives d'accès aux fichiers et aux commandes diffèrent fortement entre MySQL,...
