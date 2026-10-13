---
order: 8
title: "Injection SQL aveugle temporelle dans PostgreSQL"
description: "Déduire des données PostgreSQL à partir de délais de réponse conditionnels avec pg_sleep piloté par une expression CASE, quand le vrai et le faux se ressemblent."
keywords:
  - time based blind
  - pg_sleep
  - délai conditionnel
  - CASE WHEN
  - timing PostgreSQL
---

# Temporelle

Quand le vrai et le faux produisent la même réponse, c'est le timing qui porte le signal : une charge qui ne retarde que sur une condition vraie transforme la latence de réponse en réponse oui/non. PostgreSQL retarde avec `pg_sleep(seconds)`.

Un délai inconditionnel confirme que la technique est opérante. `pg_sleep` est une fonction renvoyant un ensemble de lignes, elle s'utilise donc depuis une clause `FROM` :

```sql
' AND 1=(SELECT 1 FROM pg_sleep(5))-- 
```

Rendez-le conditionnel en contrôlant le sleep avec `CASE`, de sorte que le délai ne se déclenche que lorsque le test est vrai :

```sql
' AND 1=(CASE WHEN (ascii(substring(current_user,1,1))>77) THEN (SELECT 1 FROM pg_sleep(5)) ELSE 1 END)-- 
```

Les deux branches du `CASE` renvoient 1, de sorte que le `1=...` englobant est toujours vrai et que la réponse est inchangée hormis le délai, qui ne survient que sur la branche vraie. Effectuez une recherche binaire sur chaque caractère exactement comme dans l'extraction booléenne, en lisant la latence au lieu du contenu.

Là où l'injection permet les requêtes empilées, une seconde instruction avec un `pg_sleep` conditionné est une alternative :

```sql
'; SELECT CASE WHEN (ascii(substring(current_user,1,1))>77) THEN pg_sleep(5) ELSE pg_sleep(0) END-- 
```

Le timing est sensible à la gigue réseau et à la charge, maintenez donc des délais de plusieurs secondes et répétez un résultat positif avant de lui faire confiance.

## Outils

- **sqlmap** : extraction aveugle temporelle automatisée avec `pg_sleep`.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- PostgreSQL Documentation: `pg_sleep`, CASE, set-returning functions
- PortSwigger Web Security Academy: Blind SQL injection
