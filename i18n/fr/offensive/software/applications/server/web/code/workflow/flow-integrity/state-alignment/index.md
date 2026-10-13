---
title: "Alignement de l'état de workflow : réconcilier l'interface, les lignes de base de données et les points de contrôle de session dans les flux multi-étapes"
description: "Garder l'interface, les enregistrements de base de données et les points de contrôle de session cohérents à travers les flux multi-étapes."
keywords:
  - état de workflow
  - session
---

# Alignement de l'état

L'alignement de l'état interroge si la **surface** (ce que voit l'utilisateur), l'**enregistrement** (la ligne faisant autorité en base de données) et la **session** (point de contrôle éphémère) décrivent tous la même étape d'un workflow. La dérive permet le saut, la soumission en double ou des transitions vers la mauvaise branche.

## Pages

| Page | Objet |
|------|--------|
| [Surface vs version](surface-version.md) | Compteur d'étape client vs version du workflow serveur |
| [Statut d'enregistrement](record-status.md) | Champs de statut de ligne vs effets de bord déjà exécutés |
| [Point de contrôle de session](session-checkpoint.md) | Clés de session qui conditionnent les étapes sans corroboration en base |

## Outils

- **Burp Suite (Repeater)** : rejouer les étapes tout en mutant l'enregistrement sous-jacent pour faire émerger la dérive entre surface, enregistrement et session.
- Tests manuels avec Burp Repeater et payloads sur mesure.

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
