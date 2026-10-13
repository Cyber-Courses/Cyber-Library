---
title: "Opérations d'objets par lot : endpoints en masse qui sautent les contrôles d'autorisation par élément"
description: API qui traitent de nombreux identifiants d'objets en une seule requête mais appliquent les contrôles d'autorisation de façon incohérente à travers le lot.
keywords:
  - API par lot
  - opération en masse
  - BOLA
  - IDOR
---

# Opérations par lot

## Contexte

Un `POST` ou `DELETE` en masse avec un tableau d'identifiants est efficace pour les clients. Le serveur peut authentifier l'appelant, valider le premier identifiant, et boucler sur le reste avec un contrôle plus faible ou manquant. Un seul appel peut franchir les frontières de tenant ou d'utilisateur.

## Théorie

Les signaux offensifs incluent des résultats mixtes : certains identifiants de la liste réussissent et d'autres échouent, une erreur unique pour tout le lot alors qu'un refus par élément serait attendu, ou un 200 avec un objet de résultat partiel qui masque quelles lignes ont été touchées.

## Pratique

### Mélanger des identifiants autorisés et d'autres utilisateurs dans un seul corps

- En labo, construisez un corps avec deux identifiants : un possédé par la session et un d'un autre utilisateur de test. Envoyez-le à un endpoint de lecture ou de mise à jour en masse et observez le comportement d'erreur par élément par rapport à un succès complet sur tous les éléments.

## Outils

- **curl**
- **Burp Suite**
