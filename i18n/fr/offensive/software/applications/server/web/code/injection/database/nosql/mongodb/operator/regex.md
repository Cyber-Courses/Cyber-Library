---
order: 4
title: "MongoDB $regex pour l'extraction ancrée et le ReDoS"
description: "L'opérateur $regex ancre des motifs pour récupérer les champs de type chaîne caractère par caractère et, avec des motifs catastrophiques, pilote un déni de service par regex."
keywords:
  - $regex
  - injection MongoDB
  - extraction de préfixe ancré
  - extraction aveugle
  - ReDoS
  - déni de service
---

# Regex

`$regex` matche un champ de type chaîne contre une expression régulière. Injecté dans un filtre, c'est l'outil le plus tranchant pour lire des données de type chaîne via un oracle booléen, et, avec un motif délibérément coûteux, un levier de déni de service.

La précondition est que le champ atteigne le filtre en tant qu'objet contrôlé par l'attaquant, de sorte que `{"$regex": "..."}` est analysé comme opérateur.

## Extraction de préfixe ancré

La technique centrale ancre un motif au début de la valeur avec `^` et l'étend un caractère à la fois. Chaque requête répond à « le champ commence-t-il par ce préfixe », et l'oracle (une connexion réussie, une redirection, un nombre de lignes, une différence de longueur de réponse) porte la réponse :

```json
{"username": "admin", "apiKey": {"$regex": "^a"}}
{"username": "admin", "apiKey": {"$regex": "^ab"}}
{"username": "admin", "apiKey": {"$regex": "^abc"}}
```

La boucle d'extraction : maintenez le préfixe confirmé, ajoutez tour à tour chaque caractère candidat, soumettez et lisez l'oracle ; le caractère qui le maintient vrai est correct, donc ajoutez-le et continuez. Via un analyseur d'objets imbriqués, la même sonde est :

```
username=admin&apiKey[$regex]=^abc
```

Restreignez le jeu de caractères pour accélérer le parcours. La correspondance insensible à la casse avec le flag `$options` divise l'alphabet par deux quand la casse n'est pas nécessaire, et une classe de caractères teste tout un groupe par requête :

```json
{"username": "admin", "apiKey": {"$regex": "^abc", "$options": "i"}}
{"username": "admin", "apiKey": {"$regex": "^abc[0-9a-f]"}}
```

Une classe hexadécimale comme `[0-9a-f]` est idéale contre les tokens et les hashs, confirmant la famille du caractère suivant en une requête avant de cerner la valeur exacte. Apprenez d'abord la longueur avec une sonde de longueur ancrée, qui ne matche que les valeurs d'au moins N caractères :

```json
{"username": "admin", "apiKey": {"$regex": "^.{32}"}}
```

Échappez les métacaractères regex (`.`, `*`, `+`, `?`, `(`, `)`, `[`, `]`, `{`, `}`, `^`, `$`, `|`, `\`) dans les caractères candidats pour que chacun soit matché littéralement plutôt que comme syntaxe. Le résultat est la récupération complète de tout champ de type chaîne atteignable par le filtre, une requête par caractère ou classe de caractères testé. C'est le moteur d'extraction derrière [Opérateurs de comparaison](comparison-operators.md) et [Opérateurs de tableau](array-operators.md).

## Déni de service ReDoS

Lorsque l'application laisse l'entrée de l'attaquant devenir le motif (plutôt que le sujet) d'un regex, une expression à backtracking catastrophique bloque le moteur de correspondance contre de longues entrées. Des quantificateurs imbriqués sur une classe se recouvrant sont la forme classique :

```json
{"field": {"$regex": "^(a+)+$"}}
{"field": {"$regex": "(a|a)*$"}}
{"field": {"$regex": "(.*a){50}"}}
```

Matchées contre une valeur stockée (ou fournie par l'attaquant) suffisamment longue qui satisfait presque le motif, ces expressions explosent en backtracking exponentiel, clouant le CPU sur le chemin de la requête et affamant les autres requêtes. Une seule requête de ce type, ou une poignée lancées en parallèle, suffit à dégrader un service. La même classe de motif est efficace partout où une entrée utilisateur atteint un regex, de sorte qu'un endpoint qui alimente le motif dans `$regex` est une primitive de déni de service directe autant qu'une primitive d'extraction.

## Outils

- **Burp Intruder** : automatiser le parcours d'extraction $regex ancré caractère par caractère.
- **nosqli** : détecter les points d'injection $regex dans les paramètres.
- **NoSQLMap** : automatiser l'extraction aveugle basée sur regex.

## Références

- [MongoDB : $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
