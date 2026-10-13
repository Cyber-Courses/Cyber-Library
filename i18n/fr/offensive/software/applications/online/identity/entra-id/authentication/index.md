---
title: "Authentification Entra"
order: 1
description: "Pénétrer un tenant Entra via la connexion : énumération des utilisateurs et du tenant, password spraying, phishing par device-code et de jetons, primary refresh tokens, et contournement de l'accès conditionnel et de la MFA."
keywords:
  - authentification Entra
  - password spraying
  - device code
  - PRT
  - accès conditionnel
---

# Authentification

Obtenir le premier jeton dans un tenant Entra est un problème de l'extérieur vers l'intérieur : apprendre quels utilisateurs existent et comment le tenant s'authentifie, puis obtenir un identifiant ou un jeton sans déclencher le verrouillage ou l'accès conditionnel. Tout ici produit un jeton d'accès ou de rafraîchissement que le reste des surfaces Entra consomme.

## Ce qui est regroupé ici

- **[Énumération](enumeration.md)** : découverte non authentifiée des utilisateurs, du tenant et de la fédération.
- **[Password spraying](password-spraying.md)** : deviner des identifiants contre les endpoints de connexion dans les limites de Smart Lockout.
- **[Phishing par device code](device-code-phishing.md)** : hameçonner le flux OAuth device-code pour obtenir des jetons.
- **[Primary refresh token](primary-refresh-token.md)** : voler et rejouer le PRT depuis un appareil joint.
- **[Accès conditionnel](conditional-access.md)** : passer outre les politiques d'accès conditionnel.
- **[Contournement de la MFA](mfa-bypass.md)** : déjouer ou enregistrer des méthodes multifacteurs.
- **[Vol de jetons](token-theft.md)** : récolter et rejouer les jetons d'accès et de rafraîchissement.

## Références

- [AADInternals](https://aadinternals.com/aadinternals/)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud : Azure authentication](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
