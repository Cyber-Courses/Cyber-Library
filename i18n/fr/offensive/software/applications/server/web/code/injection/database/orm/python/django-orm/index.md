---
title: "Injection ORM Django"
description: "Injection via raw() et extra() de Django, le curseur bas niveau, et les lookups filter() contrôlés par l'utilisateur."
keywords:
  - ORM Django
  - raw()
  - extra()
  - injection filter()
  - Python
---

# ORM Django

L'ORM de Django paramètre les querysets normaux, mais plusieurs API rouvrent l'injection lorsqu'elles sont construites à partir d'une entrée non fiable : `raw()` et le `connection.cursor()` bas niveau, l'`extra()` hérité avec ses fragments `select`/`where`, et les noms de champ ou de lookup contrôlés par l'utilisateur passés dans `filter()`.

## Pages

- **[Injection extra()](extra-injection.md)** : QuerySet.extra() insère des fragments SQL bruts dans select, where, tables et order_by, lorsque ces fragments portent de l'entrée utilisateur le résultat est une injection SQL.
- **[Injection filter()](filter-injection.md)** : Étendre des dictionnaires contrôlés par l'attaquant dans QuerySet.filter()/exclude() et faire confiance aux lookups de champ fournis par l'utilisateur expose des colonnes imprévues et de la logique booléenne...
- **[Injection SQL brute](raw-sql-injection.md)** : Comment les requêtes formatées par chaîne dans raw() et extra() de Django et l'API de curseur bas niveau deviennent de l'injection SQL malgré l'ORM, avec des charges utiles concrètes.

## Outils

- **sqlmap** : exploiter les points d'injection construits par chaîne de raw() et cursor.execute().
- **Burp Repeater et Intruder** : tester les fragments extra() et les lookups filter() contrôlés par l'utilisateur.

## Références

- Documentation Django : Performing raw SQL queries
- OWASP : SQL Injection
