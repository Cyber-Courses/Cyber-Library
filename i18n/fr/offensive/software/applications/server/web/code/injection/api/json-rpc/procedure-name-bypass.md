---
order: 1
title: "Contournement du nom de procédure JSON-RPC"
description: "Lorsque la chaîne de méthode est mappée sur une procédure serveur via une réflexion non sûre, des astuces de casse, de la contrebande de délimiteurs ou du retrait de préfixe, un appelant atteint des méthodes internes ou admin que l'allowlist croyait exclure."
keywords:
  - méthode JSON-RPC
  - dispatch par réflexion
  - allowlist de procédures
  - contournement de nom de méthode
  - méthode interne
---

# Contournement du nom de procédure

Un serveur JSON-RPC mappe la chaîne `method` de la requête sur une fonction. La façon dont il effectue ce mapping décide de ce qu'un appelant peut atteindre. Une table explicite des noms autorisés vers des fonctions spécifiques est sûre ; une recherche dynamique qui résout la chaîne contre un objet, un module ou une convention de nommage ne l'est pas, car l'appelant contrôle la chaîne et peut l'orienter vers des fonctions que le concepteur n'a jamais voulu exposer.

## Dispatch par réflexion

Le schéma dangereux résout le nom de méthode directement contre un objet hôte, par exemple `getattr(handler, method)` en Python, une recherche réflexive de méthode en Java, ou l'indexation d'un module par nom. Tout attribut ou fonction joignable de cette façon devient appelable :

```json
{"jsonrpc": "2.0", "method": "_admin_reset", "params": [], "id": 1}
```

Lorsque le dispatch est `getattr(self, request.method)`, un nom de méthode commençant par un underscore, ou tout helper interne sur l'objet handler, est invoqué même si seules les méthodes publiques étaient documentées. Des noms comme `__class__`, `__init__` ou des internes du framework peuvent aussi se résoudre, atteignant bien au-delà de l'API prévue.

## Failles de normalisation de nom

Même avec un filtre de nom, la vérification et le dispatch peuvent normaliser le nom différemment. Une denylist sensible à la casse qui bloque `admin.deleteUser` ne correspond pas à une orthographe de casse différente, tandis qu'un dispatcheur insensible à la casse résout tout de même cette orthographe vers la fonction bloquée, de sorte que la variante passe au-delà d'une denylist (et au-delà de toute vérification grossière qui normalise différemment du dispatcheur) :

```json
{"jsonrpc": "2.0", "method": "Admin.DeleteUser", "params": {"id": 104}, "id": 1}
```

Les astuces de délimiteur et de namespace exploitent le même découpage : lorsque les noms sont `namespace.method` et que l'allowlist ne vérifie que le préfixe, ou retire un préfixe avant le dispatch, un séparateur injecté ou un préfixe élaboré atteint une méthode différente de celle autorisée. Les espaces en tête ou en fin, les séparateurs alternatifs (`.`, `/`, `:`) et les caractères encodés valent chacun la peine d'être testés contre la faille entre la façon dont le nom est validé et celle dont il est résolu.

## Confirmer la faille

Énumérez les noms de méthodes en sondant : les préfixes admin et internes courants (`admin`, `internal`, `debug`, `__`), les attributs joignables par réflexion, et les variantes de casse et de séparateur de méthodes connues. Une méthode qui s'exécute, ou qui renvoie une erreur différente (une erreur d'argument plutôt que méthode introuvable) alors qu'elle devrait être bloquée, révèle que le nom s'est résolu vers une vraie fonction au-delà du filtre de nom. La conception sûre, une carte explicite chaîne vers fonction sans recherche dynamique, est la chose dont ceci sonde l'absence.

## Outils

- **Burp Intruder** : fuzzing des noms de méthodes avec des préfixes admin, internes et joignables par réflexion.
- **curl** : sondage des variantes de casse, de séparateur et d'espace d'un nom de méthode.

## Références

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
