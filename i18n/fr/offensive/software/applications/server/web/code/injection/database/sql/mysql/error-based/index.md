---
order: 4
title: "Injection SQL error-based dans MySQL"
description: "Forcer MySQL à faire fuiter les résultats de requêtes dans les messages d'erreur à l'aide des fonctions XPath EXTRACTVALUE et UPDATEXML et de la technique double-query FLOOR/RAND."
keywords:
  - injection SQL error-based
  - EXTRACTVALUE
  - UPDATEXML
  - injection double-query
  - fuite d'erreur MySQL
---

# Error-based

L'injection error-based est utile quand l'application n'affiche pas les lignes de la requête mais reflète bien le texte des erreurs de base de données. L'astuce est de faire échouer une fonction d'une manière qui intègre le résultat d'une sous-requête dans la chaîne d'erreur, puis de lire la valeur dans l'erreur affichée.

MySQL a deux canaux d'erreur fiables. Les fonctions XPath `EXTRACTVALUE()` et `UPDATEXML()` (ajoutées dans MySQL 5.1.5 et toujours présentes dans MySQL 8.0, ainsi que dans MariaDB) rejettent un XPath malformé et renvoient le texte fautif dans le message, donc envelopper une sous-requête dans une expression XPath invalide fait fuiter sa sortie. L'approche plus ancienne `FLOOR(RAND())` avec `GROUP BY` force une erreur de clé dupliquée qui contient elle aussi la valeur, et fonctionne sur les versions où les fonctions XPath ne sont pas disponibles.

Les deux fonctions XPath tronquent le texte reflété à environ 32 caractères. Les valeurs plus longues se lisent par fenêtres avec `SUBSTRING(payload, 1, 32)`, puis `SUBSTRING(payload, 32, 32)`, et ainsi de suite, en réassemblant les morceaux.

## Pages

- **[EXTRACTVALUE](extractvalue.md)** : fuite via un XPath invalide dans une fonction à deux arguments.
- **[UPDATEXML](updatexml.md)** : le même canal via la fonction à trois arguments.
- **[FLOOR double-query](floor-double-query.md)** : erreur de clé dupliquée pour les serveurs sans les fonctions XPath.

## Outils

- **sqlmap** : extraction error-based automatisée via EXTRACTVALUE, UPDATEXML et double-query.
- **ghauri** : alternative rapide avec forte évasion des WAF.

## Références

- MySQL Reference Manual: EXTRACTVALUE, UPDATEXML, XPath functions
- OWASP Testing Guide: Testing for SQL Injection
