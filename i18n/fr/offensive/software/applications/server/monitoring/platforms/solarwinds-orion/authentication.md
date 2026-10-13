---
title: "Authentification : identifiants par défaut et faibles de la console Orion"
order: 1
description: "La console web SolarWinds Orion s'authentifie avec un compte Admin (mot de passe par défaut vide ou faible) et s'intègre à Active Directory. Des identifiants par défaut, faibles ou réutilisés accordent l'accès à la console, ce qui expose l'inventaire des équipements gérés et les identifiants stockés, et l'accès administrateur atteint la configuration et les fonctionnalités à l'origine des problèmes à plus fort impact de la plateforme."
keywords:
  - solarwinds admin
  - orion login
  - default credentials
  - weak password
  - active directory
---

# Authentification

La console web Orion s'authentifie contre ses propres comptes et, couramment, contre Active Directory. Le compte intégré `Admin` avait historiquement un mot de passe par défaut vide ou faible, et les mots de passe définis par les opérateurs sont fréquemment faibles ou réutilisés ; l'intégration AD signifie que des identifiants de domaine (obtenus ailleurs au cours d'une mission) peuvent aussi permettre la connexion. L'accès à la console expose l'inventaire des nœuds gérés, une carte du parc, et les identifiants d'équipement stockés, et l'accès administratif atteint la configuration, l'API Orion et les fonctionnalités sur lesquelles reposent les vulnérabilités à plus fort impact de la plateforme. Ainsi, des identifiants par défaut, faibles ou réutilisés constituent une voie directe vers la plateforme et, à travers ses secrets stockés, vers l'environnement géré.

```bash
# the Orion Admin account (historically blank/weak default)
curl -sk -c cj https://<target>/Orion/Login.aspx --data 'Username=Admin&Password='
# AD-integrated login accepts domain credentials where configured
# spray weak/reused passwords against the console
```

## Notes d'exploitation

- Essayez le compte `Admin` avec un mot de passe vide et des mots de passe courants ; la valeur par défaut historique est faible, et les mots de passe des opérateurs sont souvent réutilisés.
- L'intégration AD signifie que des identifiants de domaine obtenus ailleurs peuvent s'authentifier auprès d'Orion ; inversement, Orion détient des identifiants qui permettent de pivoter vers le domaine, voir [Récolte d'identifiants](credential-harvesting.md).
- L'accès à la console expose l'inventaire des nœuds et les identifiants stockés ; le statut administrateur atteint l'API et la configuration à l'origine des [exploits connus](known-exploits.md).
- Lorsque les identifiants échouent, un [exploit pré-authentification](known-exploits.md) propre à la version peut contourner entièrement l'authentification.

## Références

- [SolarWinds Orion : comptes](https://documentation.solarwinds.com/)
- [Avis de sécurité SolarWinds](https://www.solarwinds.com/trust-center/security-advisories)
