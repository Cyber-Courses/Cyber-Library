---
order: 3
title: "Injection SQL basée sur les erreurs dans PostgreSQL"
description: "Forcer PostgreSQL à fuiter des résultats de requête dans les messages d'erreur via des échecs de cast de type, qui renvoient la valeur fautive en entier sans limite de longueur."
keywords:
  - injection SQL basée sur les erreurs
  - erreur de cast
  - invalid input syntax
  - fuite d'erreur PostgreSQL
---

# Basée sur les erreurs

L'injection basée sur les erreurs convient aux applications qui masquent les lignes de requête mais reflètent le texte d'erreur de la base de données. PostgreSQL n'a pas de fonctions d'erreur XPath comme `EXTRACTVALUE` de MySQL ; à la place, le canal fiable est un échec de cast de type. Convertir une chaîne en type numérique déclenche `invalid input syntax for type integer: "<value>"`, et comme le message cite la valeur qui n'a pas pu être convertie, une sous-requête placée dans ce cast fuite son résultat.

L'avantage sur le canal XPath de MySQL est que PostgreSQL ne tronque pas la valeur reflétée, de sorte qu'une ligne entière ou un dump agrégé revient dans une seule erreur plutôt que par fenêtres de 32 caractères.

## Pages

- **[Erreur de cast](cast-error.md)** : fuiter des valeurs via les échecs de conversion `CAST(... AS int)`.

## Outils

- **sqlmap** : extraction automatisée basée sur les erreurs via les échecs de cast de type.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: type casts, error messages
- OWASP Testing Guide: Testing for SQL Injection
