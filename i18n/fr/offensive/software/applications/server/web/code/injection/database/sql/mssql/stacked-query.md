---
order: 8
title: "Requêtes empilées dans une injection MSSQL"
description: "Exécuter des instructions supplémentaires après un point-virgule dans une injection SQL Server, ce que les pilotes autorisent couramment, permettant EXEC, DDL et changements de configuration."
keywords:
  - requêtes empilées
  - injection par point-virgule
  - EXEC
  - sp_executesql
  - instructions multiples MSSQL
---

# Requêtes empilées

SQL Server exécute facilement des requêtes empilées : une seconde instruction séparée par `;` s'exécute généralement, et de nombreux pilotes T-SQL (y compris les usages courants d'ADO.NET et du sqlsrv de PHP) l'autorisent. Cela transforme une injection d'apparence lecture seule en exécution d'instructions arbitraires, ce qui rend accessibles les primitives MSSQL les plus puissantes.

Ajoutez une instruction qui agit :

```sql
'; UPDATE users SET is_admin=1 WHERE username='attacker'-- 
'; EXEC xp_cmdshell 'whoami'-- 
```

L'empilement est ce qui permet l'`EXEC` de procédures stockées et étendues, le couple `sp_configure`/`RECONFIGURE` qui réactive `xp_cmdshell`, le timing avec `WAITFOR` et le DDL. Le SQL dynamique via `EXEC()` ou `sp_executesql` peut aussi être chaîné, ce qui aide à assembler des commandes à partir de valeurs extraites (la construction d'un chemin UNC pour l'out-of-band, par exemple).

La disponibilité dépend toujours du pilote et de l'API : un appel paramétré qui envoie une seule instruction rejette la seconde. Confirmez l'empilement en observant un effet de bord (une ligne modifiée, un objet créé) plutôt qu'en le supposant. Quand il est bloqué, repliez-vous sur des techniques intra-requête (union, error, blind, error-based) qui aboutissent dans l'unique instruction d'origine.

## Outils

- **sqlmap** : exploite les requêtes empilées avec `--technique=S`.
- **sqlcmd** : client officiel pour confirmer l'exécution de lots multi-instructions.

## Références

- Microsoft SQL Server Documentation: batches, EXECUTE, sp_executesql
- OWASP Testing Guide: Testing for SQL Injection
