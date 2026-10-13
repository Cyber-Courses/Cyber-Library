---
title: "Mauvais usage de l'Idempotency-Key dans les API : portée erronée, clés ignorées sur les mutations et collisions entre tenants"
description: "API qui acceptent des en-têtes Idempotency-Key mais leur donnent une mauvaise portée, les ignorent sur les verbes mutants, ou font entrer en collision des clés entre tenants."
keywords:
  - idempotence
  - API
---

# Clés d'idempotence

## Contexte

Les clients envoient `Idempotency-Key` (ou similaire) pour que les retentatives ne débitent pas deux fois. Les bugs apparaissent lorsque les clés sont ignorées sur PATCH, partagées entre utilisateurs, tronquées, ou effacées après un TTL court alors que le client réessaie encore.

## Théorie

Comportement correct : même clé + même corps → même effet ; même clé + corps différent → 409 ou erreur claire. De nombreuses implémentations sautent le hash du corps.

## Pratique

- Rejouez le même POST avec la même clé et un montant modifié dans un sandbox ; comparez les réponses et l'état du backend.

## Outils

- **curl** avec des en-têtes personnalisés
- **Burp Suite**

## Références

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic
