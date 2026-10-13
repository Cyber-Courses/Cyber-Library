---
title: "Injection par interpolation de chaîne Entity Framework : FromSqlRaw avec une chaîne interpolée"
description: Passer une chaîne interpolée C# à FromSqlRaw/ExecuteSqlRaw évalue l'interpolation avant qu'EF ne la voie, de sorte que les valeurs utilisateur deviennent du SQL littéral, contrairement à FromSqlInterpolated qui paramètre.
keywords:
  - Entity Framework
  - injection par interpolation de chaîne
  - FromSqlRaw
  - FromSqlInterpolated
  - chaîne interpolée C#
  - injection SQL
---

# Injection par interpolation de chaîne

C'est un piège subtil d'EF Core. Les chaînes interpolées C# (`$"... {x} ..."`) et les méthodes `FromSqlInterpolated`/`ExecuteSqlInterpolated` d'EF ressemblent presque à l'identique à `FromSqlRaw`, mais se comportent très différemment, et les mélanger est une source fiable d'injection SQL.

## Pourquoi cela se produit

- `FromSqlInterpolated($"... {userInput} ...")` capture la chaîne interpolée sous forme de `FormattableString`. EF convertit chaque trou `{...}` en **`DbParameter`**, sûr.
- `FromSqlRaw($"... {userInput} ...")` reçoit une **`string` simple** : l'interpolation est effectuée par le runtime C# *avant* qu'EF ne soit appelé, donc `userInput` est déjà incorporé dans le texte SQL. EF ne voit que la chaîne finale et l'exécute telle quelle, injectable.

Ainsi, la même syntaxe `$"..."` est sûre avec la méthode `Interpolated` et injectable avec la méthode `Raw`. Un développeur qui refactorise de `FromSqlInterpolated` vers `FromSqlRaw` (ou copie la mauvaise surcharge) introduit silencieusement une injection.

## Schéma vulnérable

```csharp
// Interpolated string handed to the RAW method → injectable
var q = $"SELECT * FROM Users WHERE Id = {userInput}";
var users = db.Users.FromSqlRaw(q).ToList();

// Equivalent inline footgun
db.Users.FromSqlRaw($"SELECT * FROM Users WHERE Name = '{name}'");
```

## Exploitation

Comme la valeur interpolée est du SQL littéral, exploitez-la comme une injection ordinaire dans son contexte. `Id` numérique :

```
1 OR 1=1
0 UNION SELECT Id, Username, PasswordHash FROM Users
```

`Name` entre apostrophes :

```
' UNION SELECT Id, Username, PasswordHash FROM Users --
'; EXEC xp_cmdshell 'whoami' --
```

Les requêtes empilées de SQL Server (`;`) sont disponibles sur la connexion EF, permettant la modification de données et, lorsque c'est configuré et privilégié, l'exécution de commandes. Identifier quelle surcharge est utilisée (`Raw` ou `Interpolated`) pendant la revue vous indique immédiatement si le `$"..."` est une vulnérabilité ou un paramétrage sûr.

## Outils

- **sqlmap** : extraire les données une fois qu'une chaîne interpolée atteint FromSqlRaw.
- **Burp Repeater et Intruder** : délivrer des charges utiles d'évasion et d'UNION spécifiques au contexte.

## Références

- [Documentation EF Core : Raw SQL queries, parameterization](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries#passing-parameters)
- [PayloadsAllTheThings : MSSQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md)
