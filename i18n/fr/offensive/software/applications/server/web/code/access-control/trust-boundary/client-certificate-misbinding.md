---
title: "Mauvaise liaison de certificat client TLS : mauvaise identité liée aux sessions HTTP ou au contexte amont"
description: mTLS sur le mauvais saut réseau ou mappage faible entre l'identité du certificat et les lignes de compte dans le code applicatif.
keywords:
  - mTLS
  - certificat client
  - mappage de certificat
  - liaison d'identité
---

# Mauvaise liaison de certificat client

## Contexte

Le TLS mutuel peut authentifier la *connexion*, mais l'application doit mapper l'identité présentée à **un** utilisateur ou compte de service. La mauvaise liaison correspond à : le proxy a vérifié le certificat et a transmis un en-tête que l'application lit mal, ou le même certificat mappe plusieurs tenants, ou l'application traite « n'importe quel certificat client » comme « l'utilisateur est de confiance ».

## Théorie

Le signal offensif est un certificat valide pour une identité de test A utilisé sur un chemin de code qui charge la session de l'utilisateur B parce que la table de mappage ou le nom de l'en-tête est erroné. Ce n'est pas la même chose que des en-têtes forgés sans mTLS ; le certificat est réel mais la *liaison* à l'identité applicative est erronée.

## Pratique

### Détenir deux certificats de test dans un mesh de laboratoire

- Avec deux certificats client émis pour des sujets différents, appelez la même route authentifiée et comparez quel `sub` ou identifiant de compte l'application définit. Inversez l'ordre des variables de style `ssl_client_s_dn` si la stack les expose à l'application de manière incohérente.

## Outils

- **curl** avec `--cert` et `--key`
- **OpenSSL** `s_client`
- **Burp Suite** (avec un certificat client chargé pour un test dans le périmètre)
