---
order: 4
title: "Contournement d'authentification par injection LDAP"
description: "Contourner une authentification adossée à LDAP qui construit son filtre de recherche à partir du nom d'utilisateur et du mot de passe, à l'aide du caractère générique et d'une logique booléenne injectée."
keywords:
  - contournement d'authentification LDAP
  - caractère générique userPassword
  - contournement de connexion
  - injection de filtre
---

# Contournement d'authentification

Les applications qui s'authentifient auprès d'un annuaire construisent souvent un filtre de recherche à partir des identifiants soumis et considèrent une entrée correspondante comme une connexion réussie. Lorsque les champs ne sont pas échappés, la logique du filtre peut être détournée.

Prenons une connexion qui recherche avec `(&(uid=$user)(userPassword=$pass))` et connecte l'utilisateur si une entrée correspond. Le contournement le plus simple utilise le caractère générique dans le champ mot de passe, qui correspond à n'importe quelle valeur stockée lorsque l'attribut mot de passe est présent et comparé dans le filtre :

```
user: admin
pass: *
filter: (&(uid=admin)(userPassword=*))
```

Cela renvoie l'entrée admin quel que soit le vrai mot de passe. Lorsque la correspondance se fait du côté du nom d'utilisateur, une logique injectée force un filtre toujours vrai :

```
user: *)(uid=*))(|(uid=*
filter: (&(uid=*)(uid=*))(|(uid=*)(userPassword=...))
```

Cela laisse deux filtres de premier niveau adjacents (`(&(uid=*)(uid=*))` et `(|...)`) plutôt qu'un seul filtre RFC 4515 valide, donc cela ne fonctionne que contre une bibliothèque cliente permissive qui utilise le premier filtre complet et ignore le reste ; un client qui valide la chaîne entière le rejette avec une erreur de filtre invalide. Un filtre à clause unique `(uid=$user)` est plus facile et toujours valide : `user=*` renvoie la première entrée.

La forme avec mot de passe générique est la plus fiable, et elle ne fonctionne que lorsque l'application compare le mot de passe à l'intérieur du filtre plutôt que d'effectuer un bind distinct avec le mot de passe fourni (une connexion fondée sur un bind hache et vérifie côté serveur, donc le caractère générique n'aide pas dans ce cas). Déterminez quel schéma l'application utilise en testant d'abord le caractère générique.

## Outils

- **ldapsearch** : rejouer des filtres candidats contre l'annuaire pour confirmer un contournement par caractère générique ou booléen.
- **python-ldap** : scripter des tentatives de bind et de recherche pour trouver un payload de contournement fonctionnel.
- **Burp Repeater** : forger et renvoyer des requêtes de connexion avec nom d'utilisateur/mot de passe altérés.

## Références

- RFC 4515 : LDAP search filters
- OWASP Testing Guide: Testing for LDAP Injection
