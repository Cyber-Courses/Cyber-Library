---
order: 3
title: "Omission de paramètre dans les workflows : champs optionnels qui conditionnent l'autorisation, l'état ou la logique de tarification"
description: "Déclencher des branches serveur non prévues en omettant, annulant ou retirant des paramètres optionnels dans le schéma mais utilisés pour l'autorisation ou les gates de workflow."
keywords:
  - omission de paramètre
  - paramètre optionnel
  - workflow
---

# Omission de paramètre

## Contexte

L'omission de paramètre exploite les gestionnaires multi-actions et la logique de branche par défaut : si `approve` est absent, le chemin de code peut basculer par défaut sur approve ; si `amount` est manquant, une branche d'exonération de frais peut s'exécuter. Distinct du saut d'étape (URL différente) et de la BOLA sur les identifiants (ici c'est la présence de champs, pas un identifiant erroné).

## Théorie

Le merge JSON, PATCH et l'encodage de formulaire diffèrent sur null vs manquant vs chaîne vide. Le pattern exploitable est un contrôle en deny-only ou en fall-through plutôt qu'un allow explicite : `if (body.approve === false) return deny()` suivi d'une branche approve traite un `approve` omis (qui vaut `undefined`, non `false`) comme non refusé et bascule vers l'approbation. De même `if (body.amount) chargeFee()` saute entièrement les frais lorsque `amount` est omis, et un validateur qui ne s'exécute que lorsqu'un champ est présent n'est tout simplement pas exécuté pour le champ omis.

## Pratique

### Suppression champ par champ

- Dans un proxy, retirez chaque clé JSON d'une requête modifiant l'état et comparez les réponses et les effets de bord sur des données de test.

### Matrice vide vs manquant vs null

- Envoyez `{}`, `{"approve":null}`, `{"approve":""}` sur le même endpoint et notez quelle combinaison franchit un gate.

## Outils

- **Burp Suite**
- **curl**

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
