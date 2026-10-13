---
order: 3
title: "Injection d'opérateurs de comparaison MongoDB pour contournement et extraction aveugle"
description: "Injecter $ne, $gt, $lt et $gte remplace la comparaison prévue pour contourner des filtres, et se combine à $regex pour extraire des données un caractère à la fois en aveugle."
keywords:
  - opérateurs de comparaison
  - $ne
  - $gt
  - $lt
  - injection NoSQL aveugle
  - extraction de données MongoDB
---

# Opérateurs de comparaison

Les opérateurs de comparaison (`$ne`, `$gt`, `$lt`, `$gte`, `$lte`, `$eq`) sont les chevaux de bataille de l'injection MongoDB. Injectés à la place d'un scalaire, ils réécrivent la comparaison qu'un filtre effectue, ce qui contourne les vérifications et, via un oracle vrai/faux, extrait des données sans qu'aucune ligne ne soit reflétée.

La précondition est que le champ atteigne le filtre en tant qu'objet contrôlé par l'attaquant, pas une chaîne. Une valeur convertie avec `String(input)` est matchée littéralement et rien de ce qui suit ne s'applique.

## Contourner un filtre

`$ne` (non égal) et `$gt` (supérieur à) contre une borne triviale font qu'une condition matche chaque document qui possède le champ :

```json
{"password": {"$ne": null}}
{"password": {"$gt": ""}}
{"active": {"$ne": false}}
```

`{"$ne": null}` matche toute valeur non nulle ; `{"$gt": ""}` matche toute chaîne supérieure à la chaîne vide, c'est-à-dire toute chaîne non vide. Livrés via un analyseur d'objets imbriqués, les mêmes payloads sont :

```
password[$ne]=
password[$gt]=
active[$ne]=false
```

Utilisé sur un filtre de connexion, c'est le [contournement d'authentification](../authentication-bypass.md) ; utilisé sur un endpoint de listing ou de recherche, il élargit les résultats au-delà de la ligne prévue.

## Un oracle booléen pour l'extraction aveugle

Lorsque la réponse ne reflète pas les données mais diffère tout de même entre une requête matchante et non matchante (un nombre de lignes, une redirection, un code de statut, une longueur de réponse), les opérateurs de comparaison construisent un oracle oui/non. Les champs numériques et de date tombent par recherche binaire avec `$gt`/`$lt` :

```json
{"username": "admin", "age": {"$gt": 30}}
{"username": "admin", "age": {"$lt": 40}}
{"username": "admin", "age": {"$gte": 35}}
```

Chaque requête répond à une comparaison ; divisez l'intervalle par deux à chaque étape jusqu'à cerner la valeur exacte d'un champ numérique (un âge, un solde, un timestamp, un compteur de type auto-incrément).

## Associer à $regex pour les champs de type chaîne

Les opérateurs de comparaison ordonnent les valeurs mais ne peuvent pas lire le contenu d'une chaîne, de sorte que l'extraction de chaîne associe le même oracle à `$regex`. Ancrez un motif sur un préfixe connu et étendez-le caractère par caractère, en conservant chaque caractère qui maintient l'oracle vrai :

```json
{"username": "admin", "secret": {"$regex": "^s"}}
{"username": "admin", "secret": {"$regex": "^se"}}
{"username": "admin", "secret": {"$regex": "^sec"}}
```

Une boucle typique : fixez le préfixe connu, essayez chaque caractère candidat ajouté à l'ancre, soumettez la requête et lisez l'oracle. Un résultat vrai signifie que le caractère est correct ; passez à la position suivante. Combinez avec `$gt`/`$lt` sur des champs porteurs de longueur ou avec des sondes de longueur `$regex` (`^.{10}` matche les valeurs d'au moins dix caractères) pour savoir jusqu'où extraire.

```
username=admin&secret[$regex]=^sec
```

Échappez les métacaractères regex (`.`, `*`, `+`, `?`, `(`, `)`, `[`, `]`, `\`, `^`, `$`) dans les caractères candidats pour que chacun matche littéralement. Le résultat est la récupération complète des champs de type chaîne (hash de mot de passe, tokens, réponses de récupération) à partir d'un seul signal booléen, une requête par caractère testé. Voir [Regex](regex.md) pour le détail de l'ancrage et [Exists](exists.md) pour découvrir quels champs cibler.

## Outils

- **Burp Intruder** : automatiser la recherche binaire $gt/$lt et le parcours caractère par caractère $regex.
- **nosqli** : détecter l'injection d'opérateur de comparaison dans les paramètres.
- **NoSQLMap** : automatiser l'extraction par oracle booléen.

## Références

- [MongoDB : Query comparison operators](https://www.mongodb.com/docs/manual/reference/operator/query-comparison/)
- [MongoDB : $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
