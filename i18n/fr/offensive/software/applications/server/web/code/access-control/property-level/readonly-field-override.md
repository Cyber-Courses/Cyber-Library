---
title: "Contournement d'un champ en lecture seule : PATCH et fusions qui ignorent l'immuabilité ou les champs appartenant au serveur"
description: Les métadonnées OpenAPI ou d'interface marquent un champ en lecture seule alors que le gestionnaire d'écriture copie toujours cette clé depuis le corps de la requête.
keywords:
  - champ en lecture seule
  - OpenAPI
  - overposting
---

# Contournement d'un champ en lecture seule

## Contexte

« Lecture seule » dans un client généré ou dans OpenAPI n'est pas un contrôle côté serveur. Si le DTO `UserUpdate` dans le code inclut toujours `createdAt` ou `creditScore` dans l'ensemble lié, un appel d'API direct peut les définir.

## Théorie

Identique au mass assignment avec un champ *documenté* comme immuable. La preuve offensive est un seul `PATCH` avec la clé supposée en lecture seule.

## Pratique

### Envoyer des champs documentés en lecture seule sur la route d'écriture

- En laboratoire, ajoutez `id`, `createdAt` ou un `readOnlyFlag` de la réponse `GET` dans un corps `PATCH` et voyez si le serveur accepte l'écriture et la reflète dans le `GET`.

## Outils

- **curl**
- **Burp Suite**
