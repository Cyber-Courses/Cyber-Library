---
title: "Injection SQLAlchemy"
description: "Injection via les chaînes brutes de execute() et les fragments text() construits avec des f-strings ou de la concaténation."
keywords:
  - SQLAlchemy
  - text()
  - execute()
  - SQL brut
  - Python
---

# SQLAlchemy

Le langage d'expression Core et ORM de SQLAlchemy paramètre les valeurs, mais les chaînes brutes passées à `execute()` et les fragments `text()` construits avec des f-strings ou de la concaténation sont injectables, le plus souvent lorsqu'un développeur ajoute un `WHERE`, `ORDER BY` ou `LIMIT` dynamique à une requête par ailleurs sûre.

## Pages

- **[Injection SQL brute](raw-sql-injection.md)** : passer des chaînes concaténées à Connection/Session.execute() ou Engine.execute() contourne les paramètres liés de SQLAlchemy et produit une injection SQL.
- **[Injection text()](text-injection.md)** : text() marque un fragment SQL brut ; construire ce fragment avec des f-strings ou de la concaténation plutôt qu'avec des :params bindés le rend injectable même au sein d'un code autrement OR...

## Outils

- **sqlmap** : exploitation des points d'entrée execute() bruts et text() construits par chaîne.
- **Burp Repeater** : test de l'interpolation dynamique de WHERE, ORDER BY et LIMIT.

## Références

- Documentation SQLAlchemy : Using textual SQL
- OWASP: SQL Injection
