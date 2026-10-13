---
title: "Injection Entity Framework Core"
description: "Injection via FromSqlRaw/ExecuteSqlRaw et le piège des chaînes interpolées dans EF Core."
keywords:
  - Entity Framework
  - EF Core
  - FromSqlRaw
  - interpolation de chaîne
  - .NET
---

# Entity Framework

Entity Framework Core paramètre les requêtes LINQ, mais sa famille de méthodes SQL brutes, `FromSqlRaw`, `ExecuteSqlRaw`, `SqlQueryRaw`, exécute la chaîne qu'on lui donne, et passer une chaîne interpolée C# à une méthode `*Raw` incorpore l'entrée utilisateur dans le SQL avant même qu'EF ne la voie.

## Pages

- **[Injection FromSqlRaw](fromsqlraw-injection.md)** : FromSqlRaw, ExecuteSqlRaw et SqlQueryRaw exécutent du SQL contrôlé par l'attaquant lorsque la chaîne de requête est concaténée au lieu d'être paramétrée, donnant une injection SQL d...
- **[Injection par interpolation de chaîne](string-interpolation-injection.md)** : Passer une chaîne interpolée C# à FromSqlRaw/ExecuteSqlRaw évalue l'interpolation avant qu'EF ne la voie, de sorte que les valeurs utilisateur deviennent du SQL littéral, contrairement à FromSql...

## Outils

- **sqlmap** : exploiter les points d'injection FromSqlRaw et ExecuteSqlRaw contre le backend.
- **Burp Repeater** : façonner à la main les charges utiles pour les points d'injection SQL brut et chaîne interpolée.

## Références

- Documentation Microsoft EF Core : Raw SQL queries
- OWASP : SQL Injection
