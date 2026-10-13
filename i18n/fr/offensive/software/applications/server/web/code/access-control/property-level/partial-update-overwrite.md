---
title: "Écrasement par mise à jour partielle : JSON Merge Patch et PATCH effaçant des champs imbriqués ou voisins"
description: Comportement de PATCH et de fusion JSON qui écrase des champs en lecture seule, dérivés ou financiers alors que seul un sous-ensemble de champs devrait changer.
keywords:
  - PATCH
  - JSON merge patch
  - mise à jour partielle
  - overposting
---

# Écrasement par mise à jour partielle

## Contexte

`PATCH` n'est pas uniforme d'un framework à l'autre : JSON Merge Patch, fusion superficielle personnalisée et sémantique `null` pour supprimer diffèrent. Un client peut vouloir modifier `name` tandis que la fusion écrase `balance` ou `role` si ces clés sont acceptées dans l'objet d'entrée de la fusion.

## Théorie

La même classe de mass assignment apparaît lorsque l'*opération* est « partielle » mais que la forme de l'objet est le modèle complet. `null` et une clé absente signifient souvent des choses différentes ; les attaquants testent les deux.

## Pratique

### Envoyer la forme complète du document avec une clé sensible supplémentaire

- En laboratoire, faites un `GET` de l'objet, copiez le JSON, ajoutez une clé `creditLimit` ou `status` qui devrait être réservée au serveur, et envoyez un `PATCH`. Comparez le `GET` suivant.

## Outils

- **curl**
- **Burp Suite**
