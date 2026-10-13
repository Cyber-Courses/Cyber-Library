---
title: "Contournement de la MFA : déjouer ou enregistrer des seconds facteurs"
order: 6
description: "Déjouer l'authentification multifacteur Entra : fatigue MFA, vol de SIM et de jetons, enregistrement en self-service de facteurs attaquants, et manipulation des méthodes d'authentification."
keywords:
  - contournement de la MFA
  - fatigue MFA
  - méthodes d'authentification
  - SSPR
  - vol de jetons
---

# Contournement de la MFA

Une fois un mot de passe valide, la MFA est la barrière suivante, et elle tombe de plusieurs manières : épuiser l'utilisateur par des demandes push répétées, voler un jeton qui a déjà satisfait la MFA, ou enregistrer votre propre facteur sur un compte qui n'en a encore aucun.

## Enregistrer un facteur attaquant

```bash
# when an account has no MFA method registered, enrol your own at first sign-in
# via https://aka.ms/mfasetup or the authentication-methods Graph endpoint
# (self-service registration with no existing factor is the classic takeover)
```

## Manipuler les méthodes avec Graph

```bash
# with the right role, add a phone/authenticator method to a victim and own their MFA
# POST /users/{id}/authentication/phoneMethods  (Graph)
```

## Notes d'exploitation

- **Fatigue MFA** : notifications push répétées jusqu'à ce que l'utilisateur approuve ; fonctionne là où le number-matching n'est pas imposé.
- Un **PRT** ou un **jeton de rafraîchissement** volé incarne déjà la MFA, de sorte que le rejeu de jeton contourne entièrement la demande ([vol de jetons](token-theft.md)).
- Un compte enregistré par l'attaquant (sans facteur préalable) est une prise de contrôle totale ; balayez les résultats de spray à la recherche de l'état MFA-non-enregistrée.

## Outils

- **TeamFiltration** / **TokenTactics** : capturer les jetons qui portent le claim MFA.
- **Graph / AADInternals** : enregistrer ou lire les méthodes d'authentification.

## Références

- [HackTricks Cloud : MFA bypass](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [AADInternals : MFA](https://aadinternals.com/aadinternals/)
- [dirkjanm.io](https://dirkjanm.io/)
