---
title: "Injection d'opérateur de requête MongoDB"
description: "Lorsque des objets contrôlés par l'attaquant atteignent un filtre MongoDB, les opérateurs de requête remplacent la comparaison prévue, permettant le contournement, l'extraction aveugle et le déni de service."
keywords:
  - opérateurs MongoDB
  - injection d'opérateur de requête
  - injection NoSQL
  - opérateurs de comparaison
  - $regex
  - $where
---

# Opérateur

Un filtre MongoDB exprime des conditions via des **opérateurs de requête** : des clés spéciales commençant par `$` qui indiquent au moteur comment comparer un champ. `{ "age": { "$gt": 18 } }` signifie « âge supérieur à 18 ». Lorsque des données contrôlées par l'attaquant atteignent un filtre en tant qu'objet plutôt que scalaire converti, l'attaquant fournit directement ces opérateurs, et la comparaison que le développeur avait prévue est remplacée par une de son choix.

La précondition tout au long de cette section est que l'application passe un objet contrôlé par l'attaquant dans la requête sans le convertir en chaîne, de sorte que la clé `$` injectée est analysée comme opérateur plutôt que matchée comme donnée littérale. Le JSON de la requête étalé dans un filtre, et les analyseurs de query-string ou de corps qui construisent des objets imbriqués à partir de la notation entre crochets, sont les voies courantes.

Les pages ici couvrent les familles d'opérateurs qui valent la peine d'être sollicitées : les opérateurs de comparaison (`$ne`, `$gt`, `$lt`, `$gte`) pour le contournement et l'extraction aveugle, les opérateurs de tableau (`$in`, `$nin`, `$all`, `$elemMatch`) pilotés depuis des tableaux de l'attaquant, `$exists` comme oracle de présence de champ, `$regex` pour l'extraction ancrée caractère par caractère et le ReDoS, et `$where`/`mapReduce` pour le JavaScript côté serveur là où il est activé.

## Pages

- **[Opérateurs de tableau](array-operators.md)** : injecter $in, $nin, $all et $elemMatch à partir de tableaux contrôlés par l'attaquant élargit les correspondances, énumère des valeurs candidates et sonde les champs de type tableau.
- **[Opérateurs de comparaison](comparison-operators.md)** : injecter $ne, $gt, $lt et $gte remplace la comparaison prévue pour contourner des filtres, et se combine à $regex pour extraire des données un caractère à la fois en aveugle.
- **[Exists](exists.md)** : l'opérateur $exists teste si un champ est présent, offrant un oracle vrai/faux pour cartographier le schéma des documents et inférer des données lorsqu'aucune ligne n'est reflétée.
- **[Regex](regex.md)** : l'opérateur $regex ancre des motifs pour récupérer les champs de type chaîne caractère par caractère et, avec des motifs catastrophiques, pilote un déni de service par regex.
- **[Where JavaScript](where-javascript.md)** : $where et mapReduce évaluent du JavaScript côté serveur ; là où il est activé, des chaînes JS injectées s'échappent des expressions, forcent des conditions toujours vraies et expirent...

## Outils

- **NoSQLMap** : injection d'opérateur et extraction aveugle automatisées.
- **nosqli** : scanner d'injection d'opérateur MongoDB dans les paramètres web.
- **mongosh** : valider les filtres d'opérateur directement contre une base de données.
- **Burp Suite** : intercepter et altérer les requêtes HTTP adossées à une requête de base de données.

## Références

- [MongoDB : Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
