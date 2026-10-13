---
title: "Injection text() SQLAlchemy : interpolation d'une entrée dans des clauses SQL textuelles"
description: text() marque un fragment SQL brut ; construire ce fragment avec des f-strings ou de la concaténation plutôt qu'avec des :params bindés le rend injectable même au sein d'un code autrement basé sur l'ORM.
keywords:
  - SQLAlchemy
  - injection text()
  - SQL textuel
  - paramètres liés
  - ORM Python
  - injection SQL
---

# Injection text()

`text()` enveloppe une chaîne SQL littérale pour que SQLAlchemy l'exécute, et il prend en charge les **paramètres liés** via des placeholders `:name`. L'injection survient lorsque la chaîne transmise à `text()` est assemblée à partir d'une entrée utilisateur au lieu d'utiliser ces placeholders, fréquent lorsqu'un développeur ajoute un `WHERE`, `ORDER BY` ou `LIMIT` dynamique à une requête autrement basée sur l'ORM.

## Schémas vulnérables

```python
from sqlalchemy import text

# term from the request
stmt = text(f"SELECT * FROM products WHERE name LIKE '%{term}%'")
session.execute(stmt)

# dynamic ORDER BY, cannot be a bound parameter, so often interpolated
session.execute(text(f"SELECT * FROM products ORDER BY {sort}"))
```

La forme sûre est `text("... LIKE :term")` avec `.params(term=...)` ; la valeur voyage alors comme paramètre lié.

## Exploitation

Les **contextes de valeur** se comportent comme une injection standard. Pour la chaîne `LIKE` ci-dessus :

```
%' OR '1'='1
%' UNION SELECT username, password, NULL FROM users --
```

Les **contextes `ORDER BY` / d'identifiant** ne peuvent pas être paramétrés, ils restent donc injectables même dans des bases de code disciplinées. Utilisez-les pour l'inférence et les astuces de sous-requête dans le tri :

```
(CASE WHEN (SELECT 1 FROM users WHERE username='admin' AND SUBSTRING(password,1,1)='a') THEN 1 ELSE 2 END)
```

L'usage mixte est le piège classique : une requête qui binde la plupart des valeurs avec des `:params` mais ajoute une clause interpolée reste pleinement exploitable via cette clause. Adaptez les payloads au dialecte du backend configuré (PostgreSQL/MySQL/SQLite/etc.).

## Outils

- **sqlmap** : exploitation des points d'entrée text() en contexte de valeur pour l'extraction.
- **Burp Repeater et Intruder** : diffusion de payloads vers les emplacements ORDER BY et identifiants interpolés.

## Références

- [Documentation SQLAlchemy : Using textual SQL](https://docs.sqlalchemy.org/en/20/core/connections.html#using-textual-sql)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
