---
order: 3
title: "Injection SQL error-based dans SQLite"
description: "Faire fuiter des données SQLite via des primitives qui lèvent réellement des erreurs, puisque la division par zéro renvoie NULL et que les casts convertissent silencieusement au lieu d'échouer."
keywords:
  - injection SQL error based
  - erreurs SQLite
  - fonction non définie
  - dépassement d'entier
  - conversion CAST
---

# Error-based

L'injection error-based dans SQLite demande de la prudence, car les astuces qui fonctionnent ailleurs ne lèvent pas d'erreur ici. La division par zéro renvoie `NULL` (et non une erreur), et `CAST('abc' AS INTEGER)` produit silencieusement `0`, donc aucune des deux ne fait fuiter de données. Le canal repose plutôt sur des primitives qui échouent réellement, en incorporant la valeur cible dans le texte de l'erreur.

Plusieurs opérations lèvent une erreur exploitable :

```sql
-- undefined function: the message echoes the name
' AND 1=(SELECT upper(no_such_fn()))-- 
-- integer overflow
' AND 1=abs(-9223372036854775808)-- 
-- malformed JSON (JSON1 extension)
' AND 1=(SELECT json('x'))-- 
```

Celles-ci confirment l'injection et identifient le moteur, mais SQLite n'offre pas de moyen propre et général d'insérer un résultat de requête arbitraire dans un message d'erreur comme le font l'erreur de cast de PostgreSQL ou l'erreur de conversion de SQL Server. Sa surface d'erreur est étroite et les messages n'incluent pas de façon fiable une valeur complète choisie par l'attaquant. En pratique, la sortie d'erreur sert donc ici à prouver que le point est actif (une erreur `unrecognized token`, `no such column` ou `no such function`), et les données réelles sont extraites par inférence booléenne ou time-based, qui sont les canaux fiables contre SQLite.

## Outils

- **sqlmap** : détection et extraction automatisées (`--dbms=SQLite`).
- **ghauri** : alternative rapide avec une forte évasion de WAF.
- **sqlite3** : la CLI du moteur pour reproduire localement les primitives qui lèvent des erreurs.

## Références

- Documentation SQLite : fonctions de base, JSON1, messages d'erreur
- OWASP Testing Guide: Testing for SQL Injection
