---
order: 13
title: "Contournement d'authentification via injection MSSQL"
description: "Contourner un login dont le T-SQL est construit à partir des champs nom d'utilisateur et mot de passe, à l'aide de la terminaison par commentaire et de conditions toujours vraies dans SQL Server."
keywords:
  - contournement d'authentification
  - contournement de login
  - OR 1=1
  - injection par commentaire
  - login MSSQL
---

# Contournement d'authentification

Un login qui interpole les champs soumis, tel que `SELECT * FROM users WHERE username='$u' AND password='$p'`, authentifie sur n'importe quelle ligne retournée. Une injection dans l'un ou l'autre champ peut retourner la ligne cible sans le bon mot de passe.

Mettre en commentaire la vérification du mot de passe connecte en tant qu'utilisateur choisi. Le commentaire `--` de SQL Server n'a besoin d'aucun espace final, donc `admin'--` ferme la chaîne et supprime le reste de la ligne :

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Le test du mot de passe est mis en commentaire et la ligne admin est retournée. Une condition toujours vraie retourne la première ligne quand aucun compte n'est ciblé :

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

Lorsque seul le champ mot de passe est injectable et que le nom d'utilisateur est fixé, la terminaison par commentaire ne contourne pas (elle testerait juste `password='admin'`) ; utilisez une condition qui re-sélectionne la cible pour que le `AND` final soit satisfait par un `OR` :

```
password: ' OR username='admin'--
```

Parce que `AND` est prioritaire sur `OR`, ceci s'évalue comme `(username='admin' AND password='') OR username='admin'`, retournant la ligne admin. Puisque MSSQL autorise les requêtes empilées, une injection de login peut parfois aller plus loin (par exemple `; UPDATE users SET ...`), mais le contournement lui-même ne nécessite que le retour de la ligne.

## Outils

- **sqlmap** : détection automatisée du paramètre de login injectable.
- **Burp Repeater** : élaborez à la main des payloads de login par terminaison de commentaire et par OR.

## Références

- Microsoft SQL Server Documentation : comments, SELECT, operator precedence
- OWASP Testing Guide : Testing for SQL Injection
