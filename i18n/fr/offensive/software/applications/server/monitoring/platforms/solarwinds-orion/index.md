---
title: "SolarWinds Orion : attaquer la plateforme de gestion de réseau"
order: 8
description: "SolarWinds Orion est une plateforme de gestion de réseau d'entreprise qui interroge et gère les équipements de tout le parc et stocke leurs identifiants. La surface, c'est l'authentification de la console web, les identifiants d'équipements et de réseau récoltés qu'elle détient, et les chaînes connues de contournement pré-authentification et d'exécution de code à distance de la plateforme Orion, dont chacune la transforme en un large contrôle de l'environnement géré."
keywords:
  - solarwinds
  - orion
  - npm
  - credentials
  - rce
---

# SolarWinds Orion

SolarWinds Orion est une plateforme d'entreprise largement déployée pour la gestion des réseaux et des systèmes (Network Performance Monitor et les modules qui l'entourent). Elle est de grande valeur parce qu'elle à la fois atteint et stocke les identifiants de tout le parc géré : elle interroge et configure les équipements via SNMP, SSH, WMI et des API, en détenant ces identifiants, et elle occupe une position centrale dotée d'un large accès réseau. La surface d'attaque, c'est l'authentification de la console web, la récolte des identifiants d'équipements et de réseau stockés (qui déverrouillent directement l'environnement géré), et les graves vulnérabilités connues de la plateforme Orion, y compris les chaînes de contournement pré-authentification et d'exécution de code à distance. Chacune de ces voies convertit une compromission d'Orion en un contrôle des équipements et des systèmes qu'elle gère. Orion est également la plateforme à l'origine d'une compromission majeure de la chaîne d'approvisionnement, ce qui souligne sa position centrale et sa valeur.

```bash
curl -sk https://<target>/Orion/Login.aspx | grep -ioE 'SolarWinds[^<]*'   # fingerprint
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification](authentication.md)** : identifiants par défaut et faibles de la console.
- **[Récolte d'identifiants](credential-harvesting.md)** : identifiants d'équipements et de réseau stockés.
- **[Exploits connus](known-exploits.md)** : les chaînes de contournement pré-authentification et de RCE.

## Références

- [Documentation SolarWinds](https://documentation.solarwinds.com/)
- [Avis de sécurité SolarWinds](https://www.solarwinds.com/trust-center/security-advisories)
