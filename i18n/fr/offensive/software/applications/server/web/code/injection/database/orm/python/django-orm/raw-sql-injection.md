---
title: "Injection SQL brute dans l'ORM Django : raw(), extra() et construction de chaînes avec cursor.execute()"
description: Comment les requêtes formatées par chaînes dans raw(), extra() de Django et l'API bas niveau du curseur deviennent des injections SQL malgré l'ORM, avec des payloads concrets.
keywords:
  - ORM Django
  - injection SQL brute
  - raw()
  - cursor.execute()
  - django.db.connection
  - injection SQL
---

# Injection SQL brute

L'ORM de Django paramètre les opérations de queryset normales, mais il expose aussi des **portes de sortie** qui transmettent du SQL brut à la base de données : `Model.objects.raw()` et l'API bas niveau `connection.cursor()`. Lorsque le code applicatif construit la chaîne SQL de l'une ou l'autre avec du formatage de chaînes Python au lieu de **placeholders de paramètres**, l'ORM n'apporte aucune protection et le point d'entrée est une injection SQL classique.

## Schémas vulnérables

`raw()` avec une f-string ou une interpolation `%`/`.format()` :

```python
# user_id comes straight from the request
User.objects.raw(f"SELECT * FROM auth_user WHERE id = {user_id}")
User.objects.raw("SELECT * FROM auth_user WHERE name = '%s'" % name)
```

Le curseur bas niveau avec la même erreur :

```python
with connection.cursor() as cur:
    cur.execute("SELECT * FROM auth_user WHERE username = '" + username + "'")
```

La forme sûre passe des placeholders `params=[...]`/`%s` ; la forme injectable place le texte de l'attaquant directement dans la chaîne de requête.

## Exploitation

Parce que la valeur se retrouve à l'intérieur d'une instruction SQL normale, l'injection standard s'applique. Pour un contexte numérique `id` :

```
1 OR 1=1
0 UNION SELECT id, username, password FROM auth_user --
```

Pour le contexte `name`/`username` entre guillemets simples, sortez d'abord de la chaîne :

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM auth_user --
```

Le nombre et les types de colonnes doivent correspondre au `SELECT` d'origine pour qu'un `UNION` réussisse ; énumérez avec `ORDER BY n` ou des colonnes `NULL` incrémentales comme pour toute injection basée sur union. Django tourne sur PostgreSQL, MySQL, SQLite et Oracle, adaptez donc la syntaxe des commentaires (`--`, `#`) et les fonctions de chaînes au backend utilisé.

Les requêtes empilées (stacked queries) ne sont généralement **pas** disponibles via le curseur DB-API par défaut (une instruction par `execute`), à une exception notable près : sur PostgreSQL via psycopg2 (un backend Django courant), plusieurs instructions séparées par `;` dans un seul `execute()` sont autorisées, de sorte que les requêtes empilées y sont possibles. Sur les autres backends, privilégiez `UNION` et l'inférence booléenne/temporelle plutôt que `; DROP ...`.

## Notes sur les spécificités de `raw()`

`raw()` mappe les lignes vers des instances de modèle, donc les premières colonnes doivent s'aligner sur la clé primaire et les champs du modèle pour que les lignes se mappent ; un payload `UNION` doit sélectionner les colonnes dans l'ordre du modèle, en complétant avec `NULL`. Les colonnes supplémentaires en fin peuvent être récupérées via des attributs annotés.

## Outils

- **sqlmap** : automatisation de l'extraction contre les points d'entrée raw() et cursor.execute().
- **Burp Repeater et Intruder** : diffusion de payloads de sortie de contexte et UNION spécifiques au contexte.

## Références

- [Documentation Django : Performing raw SQL queries](https://docs.djangoproject.com/en/stable/topics/db/sql/)
- [Documentation Django : SQL injection protection](https://docs.djangoproject.com/en/stable/topics/security/#sql-injection-protection)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
