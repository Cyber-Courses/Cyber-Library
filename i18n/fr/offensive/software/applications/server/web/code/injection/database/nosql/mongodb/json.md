---
title: "Injection JSON et type juggling dans les requêtes MongoDB"
description: "Une valeur attendue sous forme de chaîne arrivant comme objet ou tableau transforme les comparaisons MongoDB en injection d'opérateur ; les clés dupliquées et l'imbrication construite par l'analyseur élargissent le sink."
keywords:
  - injection JSON
  - type juggling
  - MongoDB
  - pollution de paramètres
  - clés dupliquées
  - injection d'opérateur
---

# Injection JSON

L'injection d'opérateur MongoDB est, au fond, un bug de **confusion de type**. Le développeur attend un scalaire (une chaîne ou un nombre) à une clé donnée et compare contre lui ; l'attaquant fournit un type JSON différent (un objet ou un tableau) et le sens de la requête change. Comprendre comment le type d'une valeur est choisi, et comment les analyseurs construisent ce type, est ce qui rend fiables les payloads d'opérateurs ailleurs dans ce sous-arbre.

## Une chaîne attendue, un objet fourni

Une valeur nue dans un filtre signifie une égalité. Dès l'instant où la valeur est un objet, MongoDB lit ses clés comme des opérateurs :

```json
{"user": "alice"}
{"user": {"$ne": "alice"}}
```

Le premier matche le document pour `alice`. Le second matche chaque document dont le `user` n'est pas `alice`. L'application a écrit `{"user": <input>}` avec l'intention d'une égalité ; en contrôlant le type JSON de `<input>`, l'attaquant substitue un opérateur. Tout endpoint qui étale le JSON de la requête dans une requête est exposé :

```javascript
// filter expected to be { status: "active" }
const docs = await col.find({ status: req.query.status }).toArray();
// attacker sends ?status[$ne]=active  ->  { status: { $ne: "active" } }
```

La précondition de chaque variante ci-dessous est la même : la valeur atteint la requête en tant qu'objet ou tableau analysé, sans conversion en chaîne. Un `String(input)` ou une validation de schéma qui rejette les non-chaînes referme le trou de type.

## Un tableau fourni là où un scalaire est lu

Fournir un tableau est une seconde primitive de type-juggling. Certains opérateurs et certains codes d'application se comportent différemment, ou plantent de façon instructive, lorsqu'on leur passe un tableau, et un tableau est aussi la forme de payload des opérateurs de tableau :

```json
{"role": {"$in": ["admin", "superuser", "root"]}}
{"id": {"$in": [1, 2, 3, 4, 5]}}
```

Un tableau atteignant un champ comparé avec `$in`/`$nin` permet à l'attaquant d'énumérer ou d'élargir les correspondances en une seule requête. Voir [Opérateurs de tableau](operator/array-operators.md).

## Comment les analyseurs de query-string et de corps construisent des objets imbriqués

Les attaquants contrôlent rarement du JSON brut sur les endpoints de formulaire ; l'analyseur construit l'objet pour eux. Les analyseurs qui honorent la notation entre crochets (Express `qs` et `body-parser`, les `$_GET`/`$_POST` natifs de PHP) convertissent les clés entre crochets en structures imbriquées :

```
status[$ne]=active        ->  { status: { $ne: "active" } }
tags[$in][]=a&tags[$in][]=b  ->  { tags: { $in: ["a", "b"] } }
id[$gt]=0                 ->  { id: { $gt: "0" } }
```

C'est pourquoi un payload livré en `application/x-www-form-urlencoded` ou dans une query string GET atteint la requête comme un objet opérateur identique à la forme JSON. L'injection ne requiert pas un type de contenu JSON, seulement un analyseur qui produit l'imbrication et du code qui transmet le résultat dans un filtre.

## Clés dupliquées et pollution de paramètres

Lorsque la même clé apparaît deux fois, la valeur choisie dépend de l'analyseur, et l'analyseur utilisé par le framework peut différer de celui qu'un WAF ou un validateur a inspecté. La pollution de paramètres HTTP exploite cet écart :

```
username=admin&username[$ne]=x
password=guess&password[$ne]=guess
```

Une couche peut lire la première occurrence (chaîne) et voir une requête bénigne ; la couche qui construit la requête peut conserver la dernière occurrence (un objet opérateur), de sorte que le filtre qui s'exécute est le malveillant. La même divergence apparaît dans les corps JSON avec des clés dupliquées :

```json
{"password": "guess", "password": {"$ne": null}}
```

La plupart des analyseurs JSON conservent la dernière valeur, produisant l'objet opérateur, tandis qu'un scanner qui s'est arrêté à la première clé est contourné. Sondez quelle occurrence l'emporte en envoyant une valeur connue-bonne associée à une valeur connue-mauvaise et en observant si la requête authentifie ou échoue.

## Confirmer le sink de type

Envoyez le même champ de trois façons et comparez les réponses : une chaîne simple (ligne de base), un opérateur toujours vrai (`{"$ne": null}` ou `{"$gt": ""}`), et un opérateur malformé (`{"$ne":` laissé ouvert, ou `{"$foo": 1}`). Un changement de comportement entre la chaîne et l'opérateur, ou une erreur BSON/cast issue de la forme malformée, confirme que la valeur atterrit dans la requête non typée et que les opérateurs du reste de ce sous-arbre s'appliquent.

## Outils

- **Burp Repeater** : soumettre le champ en chaîne, objet opérateur et tableau pour confirmer le sink de type.
- **nosqli** : scanner les paramètres à la recherche de points d'injection de type-juggling.
- **NoSQLMap** : automatiser la livraison de payloads d'opérateur et de tableau.

## Références

- [MongoDB : Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
