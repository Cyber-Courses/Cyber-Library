---
order: 2
title: "Injection d'ORM Python"
description: "L'ORM Django et SQLAlchemy paramètrent les requêtes ordinaires, mais les portes de sortie vers le SQL brut, les clauses text et la construction de lookups ou de filtres à partir d'entrées non fiables contournent ce binding."
keywords:
  - ORM Python
  - injection ORM Django
  - injection SQLAlchemy
  - SQL brut
  - clause text
---

# Python

Les applications web Python interrogent la base via l'ORM de Django ou SQLAlchemy, qui tous deux lient les paramètres sur leurs chemins de requête normaux. L'injection apparaît au niveau des portes de sortie délibérées et aux endroits où une requête est assemblée à partir de chaînes plutôt que d'expressions.

## Les ORM ici

- **[ORM Django](django-orm/index.md)** : `raw()` et `cursor.execute()`, les fragments select et where de `extra()`, et la construction de lookups dans `filter()`.
- **[SQLAlchemy](sqlalchemy/index.md)** : les clauses `text()` et le `execute()` brut construits par formatage de chaînes plutôt que par paramètres liés.

## Le schéma commun

Les deux ORM sont sûrs lorsque la requête est exprimée via leur API avec des paramètres liés, et vulnérables lorsqu'un développeur formate une chaîne. Les `raw()`/`extra()` de Django et le `text()` de SQLAlchemy sont les surfaces brutes explicites ; le cas plus subtil est la construction d'un filtre, d'un lookup ou d'un `ORDER BY` à partir d'une entrée, où la structure (et pas seulement une valeur) provient de l'appelant. Les identifiants (noms de tables et de colonnes) ne sont jamais paramétrables, de sorte que tout chemin d'ORM qui interpole un identifiant choisi par l'utilisateur est un point d'entrée, quel que soit le binding.

## Outils

- **sqlmap** : exploitation des points d'entrée raw()/extra() de Django et text() de SQLAlchemy.
- **Burp Repeater et Intruder** : élaboration de payloads SQL bruts et abus de lookups ou de filtres.

## Références

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Django: Performing raw SQL queries](https://docs.djangoproject.com/en/stable/topics/db/sql/)
