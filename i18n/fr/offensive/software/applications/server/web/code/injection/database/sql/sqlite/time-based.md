---
order: 9
title: "Injection SQL blind time-based dans SQLite"
description: "Inférer des données SQLite à partir de délais conditionnels construits avec des expressions lourdes telles que randomblob, puisque SQLite n'a pas de fonction SLEEP."
keywords:
  - blind time based
  - randomblob
  - pas de SLEEP
  - requête lourde
  - temporisation SQLite
---

# Time-based

SQLite n'a pas de `SLEEP` ni de `pg_sleep`, donc un délai se crée en faisant effectuer à la base un travail coûteux uniquement lorsqu'une condition est vraie. Le procédé habituel est `randomblob(n)`, qui alloue `n` octets de données aléatoires ; envelopper un gros blob dans `hex()` et une comparaison force un temps CPU mesurable.

Conditionnez l'expression lourde avec `CASE` pour qu'elle ne s'exécute que sur un test vrai :

```sql
' AND 1=(CASE WHEN (unicode(substr((SELECT password FROM users LIMIT 1),1,1))>77) THEN (SELECT 1 WHERE like('A',upper(hex(randomblob(100000000))))) ELSE 1 END)-- 
```

Quand le test est vrai, SQLite hache et compare un blob aléatoire d'environ 100 Mo (une seconde ou plus de CPU) ; quand il est faux, le `ELSE` retourne immédiatement. Ajustez la taille du blob au serveur pour qu'un test vrai ajoute un délai net et reproductible. Un grand CTE récursif est une expression lourde alternative là où `randomblob` est filtré :

```sql
' AND 1=(CASE WHEN (<cond>) THEN (WITH RECURSIVE c(x) AS (SELECT 1 UNION ALL SELECT x+1 FROM c LIMIT 5000000) SELECT count(*) FROM c) ELSE 1 END)-- 
```

Effectuez une recherche binaire sur chaque caractère à partir du délai exactement comme dans l'extraction booléenne. Parce que le délai est lié au CPU plutôt qu'un sleep fixe, il est plus bruité que `WAITFOR` ou `pg_sleep`, alors gardez le travail assez lourd pour un délai distinct et répétez un résultat positif avant de lui faire confiance. Chaque sonde charge un cœur CPU, donc utilisez la plus petite taille qui se lit encore clairement.

## Outils

- **sqlmap** : extraction time-based automatisée avec des payloads de requête lourde `randomblob` (`--technique=T`).
- **ghauri** : inférence time-based rapide avec une forte évasion de WAF.
- **Burp Repeater** : mesurez le délai conditionnel à la main pour ajuster la taille du blob.

## Références

- Documentation SQLite : randomblob, hex, expressions de table communes récursives
- PortSwigger Web Security Academy: Blind SQL injection
