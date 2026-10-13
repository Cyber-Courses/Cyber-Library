---
order: 8
title: "Contournement d'authentification via injection IBM Db2"
description: "Contourner un login dont le SQL est construit à partir des champs nom d'utilisateur et mot de passe dans IBM Db2, à l'aide de la terminaison par commentaire et de conditions toujours vraies."
keywords:
  - contournement d'authentification
  - contournement de login
  - OR 1=1
  - injection par commentaire
  - login Db2
---

# Contournement d'authentification

Un login qui interpole les champs soumis, tel que `SELECT * FROM users WHERE username='$u' AND password='$p'`, authentifie sur n'importe quelle ligne retournée. Une injection dans l'un ou l'autre champ peut retourner la ligne cible sans le bon mot de passe.

Mettre en commentaire la vérification du mot de passe connecte en tant qu'utilisateur choisi. Le commentaire `--` de Db2 supprime le reste de la ligne :

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

Une condition toujours vraie retourne une ligne quand aucun compte n'est ciblé. Db2 n'a pas de `LIMIT`, donc contraignez avec `FETCH FIRST 1 ROWS ONLY` si l'application attend une seule ligne :

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

Lorsque seul le champ mot de passe est injectable et que le nom d'utilisateur est fixé, la terminaison par commentaire ne contourne pas (elle testerait juste `password='admin'`) ; re-sélectionnez la cible pour que le `AND` final soit satisfait par un `OR` :

```
password: ' OR username='admin'--
```

Parce que `AND` est prioritaire sur `OR`, ceci s'évalue comme `(username='admin' AND password='') OR username='admin'`, retournant la ligne admin. Db2 n'autorise généralement pas les requêtes empilées via les pilotes standard, donc une injection de login ne peut pas ajouter une seconde instruction, mais le contournement par retour de ligne ne nécessite que l'unique requête.

## Outils

- **sqlmap** : confirme et fingerprint le champ de login injectable.
- Test manuel avec Burp Repeater et le client Db2 (db2 ou clpplus).

## Références

- IBM Db2 SQL Reference : comments, search conditions, FETCH FIRST
- OWASP Testing Guide : Testing for SQL Injection
