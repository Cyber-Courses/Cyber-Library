---
title: "Détecter le nombre de colonnes pour une injection UNION PostgreSQL"
description: "Trouver le nombre de colonnes qu'une requête PostgreSQL projette avec le sondage d'ordinaux ORDER BY et UNION SELECT NULL incrémental, avant d'extraire des données."
keywords:
  - détection du nombre de colonnes
  - injection ORDER BY
  - UNION SELECT NULL
  - colonnes union PostgreSQL
---

# Détection du nombre de colonnes

Un `UNION SELECT` ne s'exécute que lorsqu'il projette le même nombre de colonnes que la requête d'origine, le comptage vient donc en premier. Deux sondes le trouvent.

`ORDER BY` avec un numéro de colonne ordinal réussit jusqu'à ce qu'il dépasse le nombre réel :

```sql
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3-- 
```

La requête fonctionne tant que l'ordinal est valide et échoue avec `ORDER BY position N is not in select list` dès qu'il est trop élevé. Le dernier numéro fonctionnel est le nombre de colonnes.

La sonde `UNION SELECT NULL` ajoute des listes de `NULL` croissantes jusqu'à ce que les comptages correspondent :

```sql
' UNION SELECT NULL-- 
' UNION SELECT NULL,NULL-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`NULL` est utilisé car il est compatible avec tout type de colonne, de sorte que seul le comptage importe. Le comptage erroné échoue avec `each UNION query must have the same number of columns`. Une fois les comptages alignés, remplacez tour à tour chaque `NULL` par une valeur castée telle que `'a'::text` pour savoir quelles colonnes sont rendues sur la page ; ces positions sont les endroits où vont les données extraites. Comme PostgreSQL impose les types dans un `UNION`, gardez les colonnes non reflétées à `NULL` et castez celle reflétée en `text`.

## Outils

- **sqlmap** : automatise la détection du nombre de colonnes et l'alignement de l'union avec des casts.
- **Burp Repeater** : exécuter à la main les sondes `ORDER BY` et `UNION SELECT NULL`.

## Références

- PostgreSQL Documentation: UNION, ORDER BY, type casts
- PortSwigger Web Security Academy: SQL injection UNION attacks
