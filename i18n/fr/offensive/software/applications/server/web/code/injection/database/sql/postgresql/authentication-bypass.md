---
order: 12
title: "Contournement d'authentification via l'injection PostgreSQL"
description: "Contourner un formulaire de connexion dont le SQL est construit à partir des champs nom d'utilisateur et mot de passe dans PostgreSQL, en utilisant la terminaison par commentaire et des conditions toujours vraies."
keywords:
  - contournement d'authentification
  - contournement de connexion
  - OR 1=1
  - injection de commentaire
  - connexion PostgreSQL
---

# Contournement d'authentification

Une connexion qui interpole les champs soumis, comme `SELECT * FROM users WHERE username='$u' AND password='$p'`, authentifie sur toute ligne renvoyée. Injecter dans l'un ou l'autre champ peut lui faire renvoyer la ligne cible sans le bon mot de passe.

Commenter le contrôle du mot de passe connecte en tant qu'utilisateur choisi. Dans PostgreSQL, le commentaire `--` n'a pas besoin d'un espace final, si bien que `admin'--` ferme la chaîne et supprime le reste de la ligne :

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Le test du mot de passe est mis en commentaire, la ligne admin est donc renvoyée et l'application considère la connexion comme réussie. Une condition toujours vraie renvoie la première ligne quand aucun compte précis n'est visé :

```
username: ' OR '1'='1' LIMIT 1--
query:    SELECT * FROM users WHERE username='' OR '1'='1' LIMIT 1-- ' AND password='...'
```

Quand seul le champ mot de passe est injectable et que le nom d'utilisateur est fixe, le commenter n'aide pas (`admin'--` dans le mot de passe rend simplement le test `password='admin'`). Utilisez plutôt une condition qui re-sélectionne la cible, de sorte que le `AND password=...` final soit satisfait par un `OR` :

```
password: ' OR username='admin'--
query:    SELECT * FROM users WHERE username='admin' AND password='' OR username='admin'--'
```

Comme `AND` est prioritaire sur `OR`, ceci s'évalue comme `(username='admin' AND password='') OR username='admin'`, ce qui renvoie la ligne admin quel que soit le mot de passe. Comme PostgreSQL autorise les requêtes empilées plus souvent que MySQL, un point d'injection dans la connexion peut parfois faire plus que contourner, par exemple en ajoutant `; UPDATE users SET password=...` là où le pilote le permet, mais le contournement lui-même n'a besoin que du renvoi de la ligne.

## Outils

- **sqlmap** : détection automatisée du paramètre de connexion injectable.
- **Burp Repeater** : composer à la main les charges de terminaison par commentaire et à base de OR.

## Références

- PostgreSQL Documentation: comments, SELECT, WHERE
- OWASP Testing Guide: Testing for SQL Injection
