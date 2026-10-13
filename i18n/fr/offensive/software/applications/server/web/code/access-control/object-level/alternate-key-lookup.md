---
title: "Recherche par clé alternative : IDOR via numéros de facture, e-mails et clés métier secondaires"
description: Résolution d'utilisateurs ou de ressources par e-mail, nom d'utilisateur ou slug dans des API qui renvoient ou modifient des données sans contrôle de relation avec l'appelant.
keywords:
  - énumération d'utilisateurs
  - identifiant alternatif
  - IDOR
  - BOLA
---

# Recherche par clé alternative

## Contexte

La recherche par `?email=` ou `?handle=` est pratique et parfois publique. La forme BOLA est la même que la manipulation d'identifiant : la réponse ou la mutation s'exécute pour une clé que l'appelant ne devrait pas pouvoir cibler, et le même signal que la découverte de compte peut apparaître dans le texte d'erreur.

## Théorie

Le serveur résout une clé alternative stable en un identifiant interne puis continue sans jointure de politique avec le sujet courant. Les endpoints « search » par lot qui renvoient des objets complets pour des correspondances partielles augmentent le rayon d'impact.

## Pratique

### Interroger avec l'e-mail d'un pair dans le périmètre

- Avec deux comptes de test, appelez la recherche en utilisant l'e-mail ou le handle public de l'autre utilisateur. Comparez le statut, la taille du corps et le timing à ceux d'une clé inexistante dans la même forme de requête.

## Outils

- **curl**
- **Burp Suite**
