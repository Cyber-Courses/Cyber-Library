---
order: 2
title: "Saut d'étape dans les flux web multi-étapes : navigation forcée vers des endpoints ultérieurs sans prérequis"
description: "Invoquer un endpoint ou un état de workflow ultérieur sans avoir complété les étapes prérequises : onboarding, checkout, KYC, gates de publication."
keywords:
  - saut d'étape
  - contournement de workflow
  - navigation forcée
---

# Saut d'étape

## Contexte

Le saut d'étape est l'usage direct de l'endpoint HTTP ou de l'opération API d'une étape tardive sans les artefacts de session ou de progression que le serveur croyait exiger (KYC avant versement, expédition avant débit, 2FA avant liaison). Les tests ne sont autorisés que sur les programmes que vous pouvez attaquer.

## Théorie

Les serveurs font souvent confiance à l'ordre des étapes visible côté client et stockent des indicateurs de progression faibles (`step=3` dans un cookie, JSON `currentStage` non signé). Si la machine à états n'est pas imposée côté serveur pour chaque transition, un POST forgé vers `/api/order/finalize` réussit sans les appels préalables. Les retentatives idempotentes et les clients mobiles qui sautent des étapes d'interface optionnelles font émerger la même classe.

## Pratique

### Numéroter les étapes à partir du trafic

- Capturez un flux happy-path dans Burp. Notez chaque requête modifiant l'état. Rejouez uniquement la dernière requête avec une session neuve ; si elle réussit, le serveur n'a pas lié les prérequis à cette session.

### Navigation directe vers des liens profonds

- Ouvrez des URL de mi-parcours depuis les favoris ou l'`history` après avoir complété le flux une fois ; recommencez avec un nouveau compte qui n'a jamais complété les premières étapes.

## Outils

- **Burp Suite**
- **navigateur**

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
