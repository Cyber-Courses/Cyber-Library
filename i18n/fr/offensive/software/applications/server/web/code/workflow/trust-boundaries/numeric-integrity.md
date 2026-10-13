---
order: 1
title: "Intégrité numérique au paiement et à l'encaissement : prix cachés, quantités, frais et totaux de session périmés"
description: "Falsification de prix, de quantités ou de lignes de frais cachés lorsque le serveur recalcule les totaux de façon incohérente entre la revue du panier et la capture du paiement."
keywords:
  - falsification de prix
  - logique métier
---

# Intégrité numérique

## Contexte

Le client affiche un total ; le serveur stocke les lignes d'articles en session ou dans une commande brouillon. Si l'étape de capture fait confiance aux montants soumis par le client ou à des totaux de session périmés, le montant payé peut diverger des prix du catalogue.

## Théorie

Comparez chaque champ de frais et de taxe aux règles faisant autorité du catalogue et de la juridiction, côté serveur, au moment de la capture, et pas seulement à la création du panier.

## Pratique

- Modifiez la quantité ou le prix unitaire dans un champ caché entre la revue et le paiement sur un checkout de préproduction ; comparez le montant débité dans les journaux du prestataire de paiement.

## Outils

- **Burp Suite**

## Références

- PortSwigger Web Security Academy : Business logic vulnerabilities
- OWASP WSTG : Testing for Business Logic
