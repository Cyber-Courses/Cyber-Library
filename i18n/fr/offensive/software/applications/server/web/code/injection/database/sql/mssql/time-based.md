---
order: 9
title: "Injection SQL aveugle basée sur le temps dans MSSQL"
description: "Déduire les données SQL Server à partir de délais de réponse conditionnels avec WAITFOR DELAY, et pourquoi cela nécessite généralement un contexte d'instruction (empilé)."
keywords:
  - blind basé sur le temps
  - WAITFOR DELAY
  - délai conditionnel
  - timing MSSQL
---

# Basée sur le temps

Quand le vrai et le faux semblent identiques, c'est le timing qui porte le signal. SQL Server temporise avec `WAITFOR DELAY '0:0:5'`, qui met en pause pendant la durée indiquée.

`WAITFOR` est une instruction, pas une expression, donc il s'exécute là où une instruction est permise : une requête empilée, ou un point d'injection qui est lui-même une instruction complète. Un délai inconditionnel confirme le canal :

```sql
'; WAITFOR DELAY '0:0:5'-- 
```

Rendez-le conditionnel avec `IF`, afin que le délai ne se déclenche que sur un test vrai :

```sql
'; IF (ASCII(SUBSTRING((SELECT TOP 1 name FROM sys.sql_logins),1,1))>77) WAITFOR DELAY '0:0:5'-- 
```

Une réponse lente signifie que le point de code du caractère est supérieur à 77 ; une réponse rapide signifie qu'il ne l'est pas. Procédez à une recherche dichotomique sur chaque position exactement comme pour l'extraction booléenne, en lisant la latence au lieu du contenu.

Là où les requêtes empilées ne sont pas disponibles, `WAITFOR` ne peut pas être injecté comme instruction à part entière. Le repli est une expression coûteuse dont le coût est conditionnel, par exemple en forçant une grande jointure croisée uniquement sur une branche vraie, bien que ce soit bien moins précis que `WAITFOR`. Comme le timing est sensible à la charge et au jitter, gardez des délais de plusieurs secondes et répétez un résultat positif avant de lui faire confiance.

## Outils

- **sqlmap** : extraction automatisée en blind basé sur le temps avec `WAITFOR DELAY`.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- Microsoft SQL Server Documentation: WAITFOR, IF, control-of-flow
- PortSwigger Web Security Academy: Blind SQL injection
