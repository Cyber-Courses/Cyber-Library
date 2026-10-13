---
title: "Attaques par rejeu de webhook : callbacks d'intégration en double sans nonces, horloges ni gestionnaires idempotents"
description: "Renvoyer des callbacks HTTP vers des endpoints d'intégration dépourvus de nonce, de fenêtres temporelles ou d'idempotence résistante au rejeu."
keywords:
  - webhook
  - attaque par rejeu
---

# Rejeu de webhook

## Contexte

Les systèmes partenaires envoient en POST des événements « payment captured » ou « user provisioned ». Si le récepteur accepte le même JSON deux fois et que la signature se vérifie toujours (ou s'il n'y a pas de signature), un attaquant qui a capturé un callback peut le rejouer pour accorder des droits en double.

## Théorie

La défense en profondeur utilise un HMAC avec timestamp, des identifiants d'événement uniques stockés côté serveur, et des gestionnaires idempotents. La faiblesse se situe souvent côté récepteur, pas côté émetteur.

## Pratique

- Dans un lab, capturez un callback valide avec Burp ; rejouez-le sans modification ; puis rejouez-le avec un nouvel en-tête `Idempotency-Key` si l'application l'honore de façon incohérente.

## Outils

- **Burp Suite Repeater**

## Références

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic
