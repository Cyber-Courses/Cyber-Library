---
order: 13
title: "Contournement d'authentification via injection MySQL"
description: "Contourner un login dont le SQL est construit à partir des champs nom d'utilisateur et mot de passe, à l'aide de la terminaison par commentaire et de conditions toujours vraies dans MySQL."
keywords:
  - contournement d'authentification
  - contournement de login
  - OR 1=1
  - injection de commentaire
  - login MySQL
---

# Contournement d'authentification

Un login qui construit sa requête à partir des champs soumis, comme `SELECT * FROM users WHERE username='$u' AND password='$p'`, traite toute ligne retournée comme un login valide. Une injection dans l'un ou l'autre champ peut faire retourner la ligne cible par la requête sans le bon mot de passe.

Mettre en commentaire la vérification du mot de passe est le contournement le plus propre. Fournir `admin'-- ` comme nom d'utilisateur ferme la chaîne du nom d'utilisateur et met en commentaire le reste de la ligne :

```
username: admin'-- 
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Le test `AND password` est désormais à l'intérieur d'un commentaire, donc la requête retourne la ligne admin et l'application se connecte en tant qu'admin. Dans MySQL, le commentaire `-- ` nécessite une espace finale ; `#` est une alternative qui n'en a pas besoin, et `admin'#` fonctionne là où une espace finale est supprimée.

Une condition toujours vraie fonctionne quand aucun utilisateur précis n'est visé. `' OR '1'='1` fait correspondre le `WHERE` à chaque ligne, et `LIMIT 1` le restreint à la première (souvent le premier administrateur créé) :

```
username: ' OR '1'='1' LIMIT 1-- 
query:    SELECT * FROM users WHERE username='' OR '1'='1' LIMIT 1-- ' AND password='...'
```

Les mêmes payloads vont dans le champ mot de passe lorsque le nom d'utilisateur est fixe. Le contournement dépend uniquement du fait qu'une ligne soit retournée, donc il fonctionne quelle que soit la manière dont le mot de passe est vérifié ensuite, tant que l'application assimile un jeu de résultats non vide à une connexion réussie.

## Outils

- **sqlmap** : détection automatisée du paramètre de login injectable.
- **Burp Repeater** : forger et rejouer à la main les payloads de terminaison par commentaire et basés sur OR.

## Références

- MySQL Reference Manual: comment syntax, WHERE, LIMIT
- OWASP Testing Guide: Testing for SQL Injection
