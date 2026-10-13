---
title: "Injection de propriété imbriquée : clés de type chemin qui définissent des champs profonds sur les objets serveur"
description: Fusion profonde et clés pointées qui définissent des champs internes du modèle que la liste d'autorisation de premier niveau ne nomme pas.
keywords:
  - fusion profonde
  - clé imbriquée
  - overposting
---

# Injection de propriété imbriquée

## Contexte

La fusion JSON pour `PATCH` et l'`assign` d'ORM de dictionnaires imbriqués peuvent écrire `a.b.c` là où une liste d'autorisation plate n'a vérifié que `a`. Les types `input` de GraphQL et les champs `struct` de protobuf présentent le même risque de sémantique de fusion lorsque le serveur fusionne l'entrée utilisateur dans un objet existant côté serveur.

## Théorie

L'état dangereux est : un objet imbriqué par défaut avec des valeurs par défaut sensibles, et la fusion écrase une clé enfant que le client ne devrait pas contrôler. Certaines stacks utilisent une fusion profonde de style `lodash.merge` ; d'autres remplacent des sous-objets entiers. Le comportement dépend de la bibliothèque.

## Pratique

### Cibler les clés internes en laboratoire

- Construisez un corps `PATCH` avec un chemin d'objet imbriqué qui reflète la forme de l'ORM ou du document (`user.profile.role`) et envoyez-le avec un jeton à faibles privilèges. Comparez les lectures de l'objet avant/après.

## Outils

- **curl**
- **Burp Suite**
