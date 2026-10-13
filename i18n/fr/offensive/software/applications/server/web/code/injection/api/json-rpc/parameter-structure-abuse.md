---
order: 3
title: "Abus de la structure des paramètres JSON-RPC"
description: "Lorsque les params sont acceptés de façon incohérente comme tableau ou objet, des clés manquantes se convertissent en valeurs par défaut, d'autres chemins de code se déclenchent, ou des parseurs laxistes laissent fuir du routage via les erreurs."
keywords:
  - params JSON-RPC
  - tableau de params
  - objet de params
  - coercition de paramètres
  - valeurs par défaut
---

# Abus de la structure des paramètres

JSON-RPC autorise `params` à être soit un tableau positionnel, soit un objet nommé, et le serveur lie celui qu'il reçoit aux arguments de la procédure cible. Lorsque cette liaison est laxiste, devinant les valeurs manquantes, acceptant indifféremment les deux formes, ou convertissant les types, l'appelant contrôle les valeurs d'arguments avec lesquelles la procédure s'exécute réellement, y compris celles qui sélectionnent un chemin privilégié ou suppriment une contrainte.

## Liaison tableau contre objet

Une procédure attendant des paramètres nommés peut aussi accepter un tableau positionnel, et le mapping entre positions et noms est là où les hypothèses se brisent. Envoyer l'autre forme que celle avec laquelle le serveur a été testé peut sauter un paramètre, réordonner ceux qui touchent à la confiance, ou remplir un argument que le client n'était pas censé définir :

```json
{"jsonrpc": "2.0", "method": "createUser",
 "params": {"name": "x", "role": "admin"}, "id": 1}
```

Si la forme publique prend `params: ["x"]` et que le serveur lie un objet nommé sur la même procédure, un champ comme `role` que la forme positionnelle n'exposait jamais devient définissable. L'inverse, envoyer un tableau là où un objet était attendu, peut placer des valeurs dans les mauvais arguments.

## Clés manquantes et valeurs par défaut

Lorsqu'un champ requis est omis et que le lieur fournit une valeur par défaut plutôt que de rejeter l'appel, la valeur par défaut peut être le cas non sûr. Supprimer un champ de propriétaire, de tenant ou d'indicateur laisse la procédure s'exécuter avec une valeur par défaut permissive :

```json
{"jsonrpc": "2.0", "method": "listInvoices", "params": {}, "id": 1}
```

Si `tenantId` prend par défaut une valeur non délimitée ou d'administration lorsqu'il est absent, les params vides renvoient plus que ce que l'appelant devrait voir.

## Coercition de type et champs supplémentaires

Les lieurs laxistes convertissent les types (une chaîne là où un nombre était attendu, un objet là où un scalaire l'était) et ignorent ou absorbent les champs supplémentaires. Fournir un type inattendu peut déclencher un autre chemin de code ou une erreur qui révèle du routage interne, et des champs supplémentaires peuvent atteindre un lieur trop permissif qui les mappe sur des arguments que le schéma ne listait pas. Les objets d'erreur JSON-RPC verbeux aggravent cela en renvoyant des détails de pile ou de dispatch qui cartographient la gestion des paramètres.

## Confirmer la faille

Envoyez chaque procédure à la fois comme tableau et comme objet, omettez les champs requis pour observer les valeurs par défaut, fournissez des types inattendus et ajoutez des champs nommés supplémentaires. Un appel qui réussit avec une forme ou un champ que la forme documentée n'autorisait pas, ou qui renvoie une erreur révélatrice, montre que le lieur devine plutôt que de valider contre un schéma de paramètres fixe. La validation stricte de la forme des params par un schéma par méthode avant le dispatch est le contrôle que ceci cherche.

## Outils

- **Burp Repeater** : envoi de chaque procédure à la fois comme tableau et comme objet, et omission de champs.
- **curl** : scripting de variations de type et de champs supplémentaires de la structure des params.

## Références

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
