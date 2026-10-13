---
title: "Confusion de type de propriété : formes JSON inattendues, téléversements de fichiers et conversion par l'analyseur"
description: Forcer un champ à devenir un tableau, un objet ou un autre type JSON pour contourner un schéma qui n'a validé qu'une forme de chaîne.
keywords:
  - confusion de type
  - conversion de type JSON
  - contournement de validation
---

# Confusion de type de propriété

## Contexte

La validation s'exécute souvent sur un DTO qui attend une chaîne pour `id`. Si l'analyseur réel commence par convertir `["1","2"]` ou un objet en chaîne, ou si l'ORM mappe les types différemment, le contrôle et le sink voient des formes différentes.

## Théorie

Les constructeurs de requêtes NoSQL qui acceptent des objets opérateurs de style `{"$gt":""}` sont la confusion de type classique dans une autre couche ; en liaison JSON, la même idée est `role` en liste contre chaîne. Le test consiste à faire varier les types JSON pour la même clé.

## Pratique

### Envoyer une chaîne, un tableau et un objet pour un même champ

- En laboratoire, fixez tous les autres champs requis, puis essayez `"role": "user"`, `"role": ["admin"]` et `{"role":{"name":"admin"}}` sur la même route de mise à jour. Notez quelles formes passent la validation et ce qui est stocké.

## Outils

- **curl**
- **Burp Suite**
