---
title: "Périphériques Entra"
order: 5
description: "Abus de l'identité de périphérique Entra : enregistrement ou jonction de périphériques malveillants, jetons de provisionnement d'enrôlement en masse et falsification de jonction hybride pour obtenir des jetons d'actualisation primaires."
keywords:
  - device registration
  - Azure AD join
  - BPRT
  - hybrid join
  - PRT
---

# Périphériques

L'identité de périphérique est une information d'identification dans Entra : un périphérique enregistré ou joint obtient un **jeton d'actualisation primaire** (primary refresh token) et peut satisfaire l'accès conditionnel basé sur le périphérique. Créer une identité de périphérique que vous contrôlez, par enregistrement, jeton d'enrôlement en masse ou falsification de jonction hybride, produit un PRT et un périphérique de confiance, ce qui débloque le SSO et les applications protégées par des stratégies.

## Ce qui est regroupé ici

- **[Enregistrement de périphérique](device-registration.md)** : enregistrer un périphérique malveillant pour un PRT.
- **[BPRT](bprt.md)** : jetons de provisionnement d'enrôlement en masse pour enregistrer à grande échelle.
- **[Jonction hybride](hybrid-join.md)** : forger une identité de périphérique à jonction hybride.

## Références

- [AADInternals : enregistrement de périphérique](https://aadinternals.com/aadinternals/)
- [dirkjanm.io : identité de périphérique et PRT](https://dirkjanm.io/)
- [ROADtools (roadtx)](https://github.com/dirkjanm/ROADtools)
