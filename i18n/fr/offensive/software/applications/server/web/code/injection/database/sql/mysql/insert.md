---
order: 5
title: "Injection SQL basée sur INSERT dans MySQL"
description: "Exploiter une injection à l'intérieur d'une instruction INSERT MySQL : faire fuiter des données via les fonctions d'erreur dans VALUES et écraser des lignes avec ON DUPLICATE KEY UPDATE."
keywords:
  - injection INSERT
  - ON DUPLICATE KEY UPDATE
  - prise de contrôle de compte
  - INSERT error-based
  - injection en écriture MySQL
---

# Basée sur INSERT

L'injection atterrit souvent dans un `INSERT` plutôt que dans un `SELECT`, par exemple dans un formulaire d'inscription, de commentaire ou de journalisation. Le résultat est l'absence de lignes reflétées à lire, mais un `INSERT` expose tout de même deux primitives utiles.

Les fonctions d'erreur placées dans une valeur insérée font fuiter des données même sans reflet, car l'erreur est levée pendant la construction de la ligne. Si l'instruction est `INSERT INTO log (ip, note) VALUES ('<inj>', 'x')`, injecter une valeur de sous-requête fait échouer `EXTRACTVALUE` avec la chaîne cible dans le message :

```sql
1.2.3.4',(SELECT EXTRACTVALUE(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)))))-- 
```

La primitive la plus percutante est `ON DUPLICATE KEY UPDATE`. Elle n'a pas de clause `WHERE` : la ligne qu'elle met à jour est décidée entièrement par la ligne existante avec laquelle les valeurs insérées entrent en collision sur une clé `UNIQUE` ou `PRIMARY`. Donc pour écraser un compte précis, injectez la valeur unique propre à ce compte (ici l'email de l'admin) afin que l'insertion entre en collision avec la ligne admin, et laissez la branche `UPDATE` réécrire le mot de passe. Pour `INSERT INTO users (email, password) VALUES ('<inj>', '<hash>')` :

```sql
admin@site.com','x') ON DUPLICATE KEY UPDATE password='$2y$10$attackerhash'-- 
```

La valeur `email` `admin@site.com` duplique la ligne admin existante, donc au lieu d'ajouter un compte, l'instruction exécute `UPDATE ... SET password='$2y$10$attackerhash'` sur la ligne admin, une prise de contrôle directe. Le payload exact dépend de la liste réelle des colonnes et de la colonne qui porte la contrainte unique, toutes deux récupérées au préalable avec les techniques d'énumération.

## Outils

- **sqlmap** : détecte et exploite le paramètre INSERT injectable, y compris les fuites error-based.
- **Burp Repeater** : forger à la main les payloads `ON DUPLICATE KEY UPDATE` et de fonctions d'erreur.

## Références

- MySQL Reference Manual: INSERT, INSERT ... ON DUPLICATE KEY UPDATE
- OWASP Testing Guide: Testing for SQL Injection
