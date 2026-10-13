---
title: "Injection de replacement Sequelize : usage non sûr de replacements et valeurs de second ordre"
description: Mélanger :replacements avec la concaténation de chaînes, ou alimenter des identifiants contrôlés par l'attaquant via replacements, réintroduit l'injection SQL dans les requêtes brutes Sequelize.
keywords:
  - Sequelize
  - injection de replacement
  - replacements
  - paramètres nommés
  - injection de second ordre
  - injection SQL
---

# Injection de replacement

Les `replacements` de Sequelize substituent les jetons `:name`/`?` par des **valeurs échappées** avant l'exécution de la requête. C'est sûr pour les *valeurs*, mais le mécanisme est une substitution de chaîne, non un véritable binding de requête préparée, et plusieurs schémas le déjouent : concaténer à côté des replacements, utiliser des replacements pour des identifiants, et la réutilisation de second ordre de données stockées.

## Schémas vulnérables

**Concaténation à côté des replacements**, la partie interpolée n'est pas protégée :

```js
await sequelize.query(
  "SELECT * FROM users WHERE name = :name AND role = '" + role + "'",
  { replacements: { name } }
);
```

**Position d'identifiant/structurelle interpolée**, les `replacements` échappent les *valeurs*, donc `:col` serait rendu comme un littéral entre apostrophes (`ORDER BY 'sort'`, qui est inerte). Les identifiants tels qu'une colonne ou un emplacement `ORDER BY` ne peuvent donc **pas** être paramétrés et sont couramment **interpolés** à la place, et cette interpolation est le point d'injection :

```js
// identifier can't be a replacement, so it gets concatenated, injectable
await sequelize.query(`SELECT * FROM users ORDER BY ${req.query.sort}`);
```

**Second ordre**, une valeur stockée en toute sécurité plus tôt est ensuite concaténée dans une autre requête brute sans replacements.

## Exploitation

Pour l'emplacement `role` concaténé, injectez comme dans un contexte normal de chaîne entre apostrophes :

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

Pour l'emplacement `ORDER BY`/identifiant **interpolé**, il n'y a pas d'apostrophe à échapper, injectez donc une expression directement, rappelez-vous que cela ne fonctionne que lorsque l'emplacement est interpolé, car une valeur passée via `replacements` serait échappée en un littéral inerte entre apostrophes :

```
(CASE WHEN (SELECT 1 FROM users WHERE username='admin' AND SUBSTRING(password,1,1)='a') THEN 1 ELSE 2 END)
```

Les charges utiles de second ordre sont stockées de manière bénigne et déclenchées lorsque la requête concaténante ultérieure s'exécute ; elles contournent entièrement l'inspection au moment de l'entrée. La leçon pour l'exploitation est que la présence de `replacements` ne **garantit pas** la sécurité, inspectez chaque emplacement pour savoir s'il s'agit d'une valeur (échappée) ou d'une position structurelle/concaténée (injectable).

## Outils

- **sqlmap** : exploiter les emplacements concaténés et d'identifiant interpolé à côté des replacements.
- **Burp Repeater** : sonder chaque emplacement pour un contexte de valeur ou structurel.

## Références

- [Documentation Sequelize : Raw queries, replacements vs bind](https://sequelize.org/docs/v6/core-concepts/raw-queries/#replacements)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
