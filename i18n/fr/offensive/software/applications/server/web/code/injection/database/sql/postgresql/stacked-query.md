---
order: 7
title: "Requêtes empilées dans l'injection PostgreSQL"
description: "Exécuter des instructions supplémentaires après un point-virgule dans l'injection PostgreSQL, ce que de nombreux pilotes PostgreSQL permettent, ouvrant le DDL, les écritures et COPY depuis un seul point d'injection."
keywords:
  - requêtes empilées
  - injection par point-virgule
  - instructions multiples
  - injection DDL PostgreSQL
---

# Requêtes empilées

Une requête empilée ajoute une seconde instruction indépendante après un point-virgule. Les pilotes PostgreSQL le permettent bien plus souvent que les pilotes MySQL courants : le protocole de requête simple de `libpq` et PDO sans instructions préparées exécutent chaque instruction séparée par `;`, si bien qu'un seul point d'injection peut exécuter du SQL arbitraire, pas seulement lire.

Là où le point d'insertion est un `SELECT`, ajoutez une instruction qui agit :

```sql
'; UPDATE users SET password='$2a$...' WHERE username='admin'-- 
'; CREATE TABLE x(d text)-- 
```

L'exécution empilée est ce qui rend atteignables les primitives PostgreSQL les plus puissantes depuis une requête d'apparence en lecture seule : créer une table de transit et y `COPY`-er un fichier, écrire un web shell avec `COPY ... TO` et exécuter une commande OS avec `COPY ... FROM PROGRAM` nécessitent tous une instruction propre, que l'empilement fournit.

La disponibilité dépend du pilote. Les interfaces à instructions préparées et les pilotes qui envoient une instruction par requête rejettent la seconde instruction, l'empilement se confirme donc en observant un effet de bord (une table créée, une ligne mise à jour) plutôt qu'en le supposant. Quand il est bloqué, repliez-vous sur les techniques intra-requête (union, erreur, aveugle) qui fonctionnent au sein de l'unique instruction d'origine.

## Outils

- **sqlmap** : exploite les requêtes empilées avec `--technique=S`.
- **psql** : client officiel pour confirmer l'exécution multi-instructions.

## Références

- PostgreSQL Documentation: multi-statement simple query, COPY
- OWASP Testing Guide: Testing for SQL Injection
