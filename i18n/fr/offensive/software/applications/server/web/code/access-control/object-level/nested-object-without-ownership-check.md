---
title: "Accès à une ressource imbriquée sans contrôle de propriété : IDOR sur des URL parent/enfant"
description: Chemins hiérarchiques où le parent est autorisé mais où il n'est pas vérifié que l'identifiant enfant appartient à ce parent.
keywords:
  - ressource imbriquée
  - BOLA
  - graphe d'objets
---

# Accès à un objet imbriqué

## Contexte

Des chemins comme `/orgs/5/invoices/99` peuvent vérifier l'appartenance à l'org 5 mais ne joignent jamais `invoices` à cette org, de sorte que 99 d'une autre org est tout de même renvoyé. La défaillance est un invariant manquant sur la clé complète (org, enfant), non un simple identifiant sur une table unique.

## Théorie

L'« include » ORM d'enfants imbriqués et les resolvers GraphQL qui chargent par lots en se basant uniquement sur l'identifiant d'enfant sont des formes de code courantes. Le test offensif consiste à détenir un identifiant d'enfant valide du tenant A et à le placer sous le parent du tenant B dans le chemin.

## Pratique

### Échanger l'identifiant enfant entre parents en labo

- Créez des objets enfants sous l'org A. Notez un identifiant d'enfant. Utilisez l'org B dans le chemin avec l'identifiant d'enfant de A. Un `200` est un signal fort d'un contrôle de jointure manquant.

## Outils

- **Burp Suite**
- **curl**
