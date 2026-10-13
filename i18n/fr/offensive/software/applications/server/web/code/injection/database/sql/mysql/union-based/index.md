---
order: 10
title: "Injection SQL basée sur UNION dans MySQL : alignement des colonnes et extraction de données"
description: "Utiliser UNION SELECT dans MySQL pour ajouter des lignes choisies par l'attaquant au résultat d'une requête, de la détection du nombre de colonnes à l'extraction via information_schema."
keywords:
  - injection SQL basée sur union
  - UNION SELECT MySQL
  - détection du nombre de colonnes
  - extraction via information_schema
  - GROUP_CONCAT
---

# Basée sur UNION

Lorsqu'une requête injectable renvoie ses lignes à la page, un `UNION SELECT` vous permet d'ajouter des lignes de votre choix. Le `SELECT` ajouté s'exécute avec les privilèges de l'application et ses colonnes apparaissent là où le résultat d'origine est affiché, ce qui transforme une requête renvoyant des données en une primitive de lecture générale sur tout ce que le compte de base de données peut atteindre.

Deux conditions doivent être réunies. Le `SELECT` injecté doit projeter le **même nombre de colonnes** que la requête d'origine, et les colonnes que vous voulez lire doivent se trouver à des positions dont les **types sont compatibles** avec ce que la page restitue (généralement une colonne de chaîne). `NULL` est compatible avec tout type, c'est donc le remplissage sûr pendant que vous déterminez la disposition.

Le déroulement est le suivant : détecter le nombre de colonnes, trouver quelles colonnes sont reflétées, puis lire le schéma et les données à travers ces positions. Le `information_schema` de MySQL fournit le schéma, et `GROUP_CONCAT()` regroupe de nombreuses lignes en une seule afin qu'une unique cellule reflétée puisse porter toute une table.

## Pages

- **[Détecter le nombre de colonnes](detect-column-count.md)** : sondage `ORDER BY` et `UNION SELECT NULL`.
- **[Extraire avec information_schema](extract-with-information-schema.md)** : énumérer et récupérer via le catalogue.
- **[Extraire sans information_schema](extract-without-information-schema.md)** : quand le catalogue est filtré.

## Outils

- **sqlmap** : détection et extraction automatisées basées sur UNION.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- MySQL Reference Manual : UNION clause and `information_schema`
- OWASP Testing Guide : Testing for SQL Injection
