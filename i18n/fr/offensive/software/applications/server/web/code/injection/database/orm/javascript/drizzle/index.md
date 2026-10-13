---
order: 3
title: "Injection Drizzle"
description: "Le template sql de Drizzle lie ses interpolations sous forme de paramètres et ses fonctions d'opérateur sont sûres côté valeur, donc l'injection apparaît via sql.raw et les fragments bruts, et via les filtres dont la colonne ou l'opérateur provient de l'entrée."
keywords:
  - Drizzle
  - sql.raw
  - template sql Drizzle
  - injection d'opérateur
  - fragment brut
---

# Drizzle

Drizzle est paramétré sur ses chemins normaux : le template taggé `sql` lie chaque interpolation, et les helpers d'opérateur (`eq`, `gt`, `inArray`, et les autres) passent les valeurs sous forme de paramètres. L'injection apparaît là où le code descend à un fragment brut, ou là où la structure d'un filtre, sa colonne ou son opérateur, est choisie à partir d'une entrée non fiable.

## Où ça dérape

- **[Injection SQL brute](raw-sql-injection.md)** : `sql.raw()` et les chaînes concaténées à l'intérieur d'un fragment atteignent la base de données sans paramétrage.
- **[Injection d'opérateur](operator-injection.md)** : sélectionner une colonne ou un opérateur à partir de l'entrée, ou étaler un filtre non fiable, change les lignes qui correspondent au-delà du prédicat prévu.

## Les chemins sûrs et non sûrs

Le template `sql` est la primitive sûre : `sql\`... ${value} ...\`` lie `value`. L'échappatoire est `sql.raw(string)`, qui insère la chaîne littéralement, destinée au SQL dynamique de confiance et dangereuse dès l'instant où la chaîne porte de l'entrée. Les fonctions d'opérateur sont sûres pour les valeurs qu'elles comparent, mais Drizzle permet de composer une requête dynamiquement, donc le code qui décide quelle colonne ou quel opérateur utiliser à partir de la requête remet à l'appelant une partie de la forme de la requête. Le template et les valeurs d'opérateur sont sûrs ; `sql.raw`, la concaténation à l'intérieur d'un fragment, et les colonnes ou opérateurs choisis depuis l'entrée sont les points d'injection.

## Outils

- **sqlmap** : exploiter les points d'injection sql.raw et de fragment concaténé.
- **Burp Repeater** : tester les colonnes, opérateurs et fragments bruts choisis depuis l'entrée.

## Références

- [Drizzle : Magic sql operator](https://orm.drizzle.team/docs/sql)
- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
