---
order: 5
title: "Injection LDAP en aveugle"
description: "Déduire les valeurs d'attributs d'annuaire caractère par caractère grâce à la correspondance par caractère générique LDAP et aux différences de réponse vrai/faux lorsqu'aucune donnée n'est renvoyée."
keywords:
  - injection LDAP en aveugle
  - inférence par caractère générique
  - extraction d'attributs
  - oracle booléen
---

# Aveugle

Lorsqu'un filtre injectable ne renvoie pas les données correspondantes mais que la réponse diffère entre une correspondance et une absence de correspondance, les attributs LDAP peuvent tout de même être lus un caractère à la fois en utilisant le caractère générique comme test de préfixe.

L'oracle demande « cet attribut commence-t-il par ces caractères ? » en ancrant un caractère générique. Contre `(&(objectClass=user)(cn=$input))`, injectez un attribut cible avec un préfixe connu grandissant :

```
input: admin)(description=S*
filter: (&(objectClass=user)(cn=admin)(description=S*))
```

Une correspondance (la réponse « trouvé ») signifie que l'attribut `description` de `admin` commence par `S` ; une absence de correspondance signifie qu'il ne commence pas ainsi. Étendez le préfixe un caractère à la fois (`Se*`, `Sec*`, ...) pour récupérer la valeur entière. L'alphabet est parcouru à chaque position, donc c'est l'analogue LDAP de l'extraction booléenne en SQL.

Les tests de présence énumèrent quels attributs existent, ce qui restreint ce qu'il faut extraire :

```
input: admin)(mail=*
filter: (&(objectClass=user)(cn=admin)(mail=*))
```

Une correspondance signifie que `admin` possède un attribut `mail`. L'injection LDAP en aveugle ne fonctionne que sur les attributs que le compte lié peut lire et qui ne sont pas en écriture seule : un `userPassword` haché n'est généralement ni renvoyé ni comparable de cette façon, mais les attributs descriptifs, de contact et de groupe le sont habituellement. Comme l'injection SQL en aveugle, c'est lent et presque toujours automatisé, mais la comparaison par caractère générique unique est la primitive fiable sous-jacente.

## Outils

- **ldapsearch** : vérifier les correspondances de préfixe par caractère générique qui alimentent l'oracle booléen.
- **Burp Intruder** : automatiser l'extraction par caractère à travers l'alphabet.
- **python-ldap** : scripter la boucle d'inférence caractère par caractère.

## Références

- RFC 4515 : LDAP search filters (substring and presence matching)
- OWASP Testing Guide: Testing for LDAP Injection
