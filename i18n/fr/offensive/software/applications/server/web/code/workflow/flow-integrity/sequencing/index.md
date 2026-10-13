---
title: "Attaques sur le séquencement des workflows : saut d'étape, accès hors ordre, approbations, omission et annulations"
description: "Abus des workflows ordonnés : sauter des étapes, appeler des étapes hors ordre, contourner des approbations, omettre des paramètres, et chemins de renouvellement ou d'annulation."
keywords:
  - saut d'étape
  - contournement de workflow
  - contournement d'approbation
---

# Séquencement

Le **séquencement** est l'axe de la **chronologie** de l'intégrité du flux : ce qui doit se produire avant quoi, quelles approbations humaines ou système conditionnent les transitions, et comment les chemins de renouvellement ou d'annulation interagissent avec la capture et le règlement.

## Pages

| Page | Objet |
|------|--------|
| [Saut d'étape](step-skip.md) | Navigation forcée vers des étapes ultérieures |
| [Accès hors ordre aux étapes](out-of-order-step-access.md) | Revisiter des étapes intermédiaires |
| [Abus d'omission de paramètre](parameter-omission-abuse.md) | Champs optionnels qui conditionnent la logique |
| [Contournement d'approbation](approval-bypass.md) | Maker-checker et approbations |
| [Contournement du renouvellement de droits](entitlement-renewal-bypass.md) | Essais, renouvellements, lacunes de revérification |
| [Abus des annulations et remboursements](reversal-and-refund-workflow-abuse.md) | Remboursements, chargebacks, races de règlement |

## Outils

- **Burp Suite (Repeater)** : rejouer et réordonner les étapes capturées pour provoquer des bugs de séquencement.
- Tests manuels avec Burp Repeater et payloads sur mesure.

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
