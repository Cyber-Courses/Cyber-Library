---
order: 1
title: "Étapes de workflow hors ordre : revisiter des états intermédiaires après des transitions ultérieures (bugs de rejeu et de branche)"
description: "Revisiter ou répéter des étapes intermédiaires de workflow après que des étapes ultérieures ont progressé, brisant les hypothèses de progression monotone dans le code serveur."
keywords:
  - workflow
  - hors ordre
  - rejeu
---

# Accès hors ordre

## Contexte

L'accès hors ordre signifie que l'utilisateur revient à une étape antérieure après que l'état ultérieur a progressé (bouton précédent du navigateur, un POST à mi-parcours mis en favori, ou une requête intermédiaire rejouée) et que le serveur accepte des changements qui invalident des engagements ultérieurs tels qu'un prix, une réservation de stock ou une approbation. C'est distinct du saut d'étape (sauter jusqu'à la fin sans contexte) : ici le contexte existe, mais l'ordre est faux.

## Théorie

Les patterns faibles sont les indicateurs de session qui ne font qu'incrémenter, l'absence de jeton de version par formulaire, les retentatives POST idempotentes qui réappliquent des remises, et l'absence de graphe côté serveur des transitions autorisées. Lorsque la progression est supposée monotone mais n'est pas imposée, le gestionnaire d'une étape antérieure continue de muter un état dont les étapes ultérieures dépendaient déjà.

## Pratique

### Bouton précédent après un changement de prix

- Avancez jusqu'à une étape de récapitulatif affichant le total `T`. Utilisez le bouton précédent du navigateur pour modifier le panier, ajouter des articles et revenir en avant. Si `T` n'est pas recalculé ni verrouillé à la validation, la commande se finalise au total périmé ; capturez l'incohérence dans un lab.

### Rejeu d'un POST à mi-parcours

- Capturez `POST /checkout/shipping`, puis finalisez le paiement. Rejouez le POST d'expédition pour changer l'adresse après la capture. Si le serveur l'accepte, la commande livrée diverge de la commande payée.

## Outils

- **Burp Suite**
- **navigateur**

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
