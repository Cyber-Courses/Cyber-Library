---
title: "Requêtes HTTP parallèles contre l'état de workflow : races sur le checkout, les transferts et les opérations conditionnées par étape"
description: "Fan-out délibéré de requêtes HTTP pour déclencher des races sur la session, le panier ou des opérations conditionnées par étape."
keywords:
  - race condition
  - requêtes parallèles
---

# Requêtes parallèles

## Contexte

Les attaquants envoient de nombreuses requêtes simultanées contre la même session ou le même identifiant de ressource pour gagner des races avant que les verrous ou les files côté serveur ne sérialisent le travail. Les cibles courantes incluent le checkout, les transferts, la promotion de rôle et l'émission de tickets.

## Théorie

L'efficacité dépend de la taille du pool de connexions, du threading du serveur applicatif, et de la question de savoir si le datastore impose des contraintes d'unicité.

## Pratique

- Dans le périmètre, scriptez 20 à 50 POST parallèles vers un endpoint de staging et comparez le nombre de lignes et les logs d'audit avec le comportement en mono-thread.

## Outils

- **Burp Suite** Turbo Intruder
- **Python asyncio** ou **GNU parallel** avec curl

## Références

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic
