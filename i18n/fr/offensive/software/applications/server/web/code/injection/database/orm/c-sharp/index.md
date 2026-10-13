---
title: "Injection ORM C#"
description: "Entity Framework et EF Core paramètrent les requêtes LINQ, mais les API SQL brutes et l'interpolation de chaîne non sûre dans FromSqlRaw et points d'injection similaires atteignent la base de données sans paramétrage."
keywords:
  - ORM C#
  - injection Entity Framework
  - EF Core
  - FromSqlRaw
  - interpolation de chaîne
---

# C#

L'accès aux données en .NET passe par Entity Framework et EF Core. Les requêtes LINQ se traduisent en SQL paramétré et sont sûres ; les API SQL brutes sont l'endroit où entre l'injection, et l'interpolation de chaîne C# fait ressembler la forme non sûre à s'y méprendre à la forme sûre.

## L'ORM ici

- **[Entity Framework](entity-framework/index.md)** : `FromSqlRaw` et `ExecuteSqlRaw` alimentés par des chaînes concaténées ou interpolées, contre la forme paramétrante `FromSqlInterpolated`.

## Le schéma commun

EF Core propose deux méthodes brutes qui se lisent presque à l'identique : `FromSqlInterpolated` capture une chaîne interpolée sous forme de `FormattableString` paramétrée et est sûre, tandis que `FromSqlRaw` prend une `string` simple et l'exécute telle quelle. Un développeur qui construit une chaîne interpolée `$"..."` et la passe à `FromSqlRaw` (ou y concatène) a écrit du code injectable qui ressemble à l'appel sûr. Les identifiants sont le seul cas où la méthode sûre ne peut pas aider, mais elle échoue de manière fermée : `FromSqlInterpolated` transforme chaque trou en `DbParameter`, et comme un paramètre ne peut pas tenir lieu de nom de table ou de colonne, un identifiant interpolé fait rejeter la requête par la base de données plutôt que l'exécuter. L'injection d'identifiant nécessite donc `FromSqlRaw`, la concaténation ou une chaîne préformatée. L'indice est `Raw` alimenté par quoi que ce soit que l'appelant a influencé, ou un identifiant intégré à la main dans le texte SQL.

## Outils

- **sqlmap** : automatiser l'exploitation des points d'injection SQL brut d'EF Core.
- **Burp Repeater** : façonner les charges utiles pour les points d'injection FromSqlRaw et chaîne interpolée.

## Références

- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Microsoft : Raw SQL queries in EF Core](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries)
