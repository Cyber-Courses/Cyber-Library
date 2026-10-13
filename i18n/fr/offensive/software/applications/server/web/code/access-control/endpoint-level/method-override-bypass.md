---
title: "Contournement par method override HTTP : X-HTTP-Method-Override et verbes à travers des passerelles restrictives"
description: Frameworks qui respectent X-HTTP-Method-Override ou _method= permettant à un POST d'usurper DELETE ou PUT au-delà de règles de passerelle naïves.
keywords:
  - X-HTTP-Method-Override
  - method tunneling
  - contournement de verbe
  - autorisation REST
---

# Method override

## Contexte

Certaines stacks mappent un `POST` entrant vers `DELETE` ou `PUT` quand un en-tête ou un champ de formulaire le demande, pour les clients qui ne peuvent pas émettre certains verbes. Si le routage ou l'autorisation s'appuie sur le verbe * du fil* tel que vu par le saut extérieur mais que l'application interne réécrit le verbe effectif, une route qui était « POST uniquement » dans l'esprit de la passerelle peut tout de même exécuter une méthode qui modifie l'état sur l'application.

## Théorie

La divergence se situe entre la méthode que voit la politique de bord et la méthode qu'utilise le matcher de route après l'override. Le contournement apparaît lorsqu'une couche utilise `POST` et qu'une autre applique `DELETE` avec le même chemin et le même corps.

## Pratique

### Envoyer un POST avec en-tête d'override en labo

- Émettez un `POST` vers un chemin qui renvoie `405` pour `DELETE` au bord, avec `X-HTTP-Method-Override: DELETE` et la même forme de corps qu'un vrai `DELETE` utiliserait, et observez le statut de l'application et les effets de bord sur un objet de test.

## Outils

- **curl**
- **Burp Suite**
