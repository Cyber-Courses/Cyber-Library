---
title: "Exposition de panneaux d'administration : URL par défaut, identifiants faibles et interfaces de gestion divulguées"
description: Interfaces de gestion et consoles exposées sur des chemins courants ou protégées seulement par des identifiants par défaut, permettant des changements d'état à fort impact dans le périmètre.
keywords:
  - panneau d'administration
  - console de gestion
  - identifiants par défaut
  - console web
---

# Panneaux d'administration

## Contexte

Les panneaux d'administration regroupent la gestion des utilisateurs, la configuration et l'export de données dans une interface de navigateur. La valeur offensive est élevée lorsque le chemin est devinable, que l'installation utilise encore les identifiants par défaut, ou que le panneau est exposé à l'internet ouvert alors que le produit supposait un VLAN de gestion.

## Théorie

Les chemins typiques incluent `/admin`, `/wp-admin`, les URL de console spécifiques au produit, et les interfaces d'administration de base de données laissées sous la racine de l'application. Le succès est souvent une combinaison de découverte de chemin, de listes de mots de passe par défaut, et du comportement de session après connexion (portée du cookie, absence de MFA sur le seul panneau).

## Pratique

### Cartographier le modèle d'authentification du panneau en labo

- Après connexion, identifiez si la session est la même que celle de l'application principale ou un cookie au nom distinct. Testez si une session applicative à faibles privilèges peut faire un `GET` sur les chemins d'administration sans second facteur quand le produit prétend que la MFA est activée.

## Outils

- **Burp Suite**
- **browser**
