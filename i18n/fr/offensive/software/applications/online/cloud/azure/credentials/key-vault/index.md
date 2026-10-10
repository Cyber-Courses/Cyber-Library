---
title: "Key Vault"
order: 2
description: "Lire le matériel d'Azure Key Vault : secrets, clés et certificats, atteints via un accès au plan de données ou un octroi par stratégie d'accès ou RBAC."
keywords:
  - Key Vault
  - secrets
  - clés
  - certificats
  - stratégie d'accès
---

# Key Vault

Azure Key Vault est l'endroit où les applications stockent leurs mots de passe, chaînes de connexion, clés de signature et certificats TLS, de sorte qu'un coffre que vous pouvez lire est une ligne directe vers les secrets de l'environnement. L'accès est régi soit par des **stratégies d'accès** héritées, soit par **Azure RBAC**, tous deux sur le plan de données du coffre à `https://<vault>.vault.azure.net` ; un jeton d'identité managée pour `https://vault.azure.net` ou un utilisateur doté d'un rôle de plan de données le lit.

## Ce qui s'y rattache

- **[Secrets](secrets.md)** : lister et déverser les valeurs des secrets stockés.
- **[Clés et certificats](keys-and-certificates.md)** : extraire ou utiliser des clés et télécharger des certificats avec leur clé privée.
- **[Stratégie d'accès](access-policy.md)** : s'octroyer l'accès au coffre quand on détient des droits de plan de gestion mais pas de plan de données.

## Références

- [Microsoft : accès au plan de données Key Vault](https://learn.microsoft.com/azure/key-vault/general/security-features)
- [HackTricks Cloud : Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst (Key Vault)](https://github.com/NetSPI/MicroBurst)
