---
title: "Détecter le nombre de colonnes pour une injection UNION MySQL"
description: "Trouver le nombre de colonnes qu'une requête MySQL projette, via le sondage ORDER BY par ordinal et l'ajout incrémental de UNION SELECT NULL, avant d'extraire les données."
keywords:
  - détection du nombre de colonnes
  - injection ORDER BY
  - UNION SELECT NULL
  - colonnes union MySQL
---

# Détection du nombre de colonnes

Un `UNION SELECT` ne s'exécute que s'il projette exactement autant de colonnes que la requête d'origine. La première étape de toute attaque union est donc de compter ces colonnes. Deux sondes le permettent.

La méthode `ORDER BY` demande à la base de données de trier selon un numéro de colonne ordinal et guette le point où le numéro dépasse le nombre réel de colonnes :

```sql
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3-- 
```

Le tri réussit tant que l'ordinal est valide et échoue avec `Unknown column '4' in 'order clause'` une fois qu'il dépasse la dernière colonne. Le plus grand nombre qui fonctionne encore est le nombre de colonnes. C'est discret (pas de lignes supplémentaires) et cela fonctionne même quand la page n'affiche qu'un statut, pas des données.

La méthode `UNION SELECT NULL` ajoute des listes croissantes de `NULL` jusqu'à ce que le nombre de colonnes concorde et que la requête s'exécute sans erreur de type ou de nombre :

```sql
' UNION SELECT NULL-- 
' UNION SELECT NULL,NULL-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`NULL` est utilisé parce qu'il est compatible avec tout type de colonne, de sorte que seul le nombre, pas les types, décide du succès. La longueur de liste qui cesse de produire `The used SELECT statements have a different number of columns` est la réponse, et la même requête vous indique déjà que l'union est active.

Une fois le nombre connu, remplacez les `NULL` un par un par un marqueur tel qu'un nombre ou une chaîne pour savoir quelles positions sont renvoyées sur la page. Ces positions reflétées sont là où les données extraites doivent aller.

## Outils

- **sqlmap** : automatise la détection du nombre de colonnes et l'alignement de l'union.
- **Burp Repeater** : exécutez les sondes `ORDER BY` et `UNION SELECT NULL` à la main.

## Références

- MySQL Reference Manual : UNION clause, ORDER BY
- PortSwigger Web Security Academy : SQL injection UNION attacks
