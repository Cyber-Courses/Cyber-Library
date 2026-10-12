---
title: "LibreNMS et Observium : attaquer les plateformes de monitoring réseau"
order: 5
description: "LibreNMS et Observium sont des plateformes de monitoring réseau en PHP qui interrogent les équipements via SNMP et stockent leurs identifiants. La surface est l'authentification web, les voies d'injection de commandes authentifiées qui atteignent l'exécution de code sur l'hôte de monitoring, et la récolte des chaînes SNMP et des identifiants d'équipements stockés qui donnent accès à tout le réseau surveillé."
keywords:
  - librenms
  - observium
  - snmp
  - command injection
  - credentials
---

# LibreNMS et Observium

LibreNMS (un fork activement développé d'Observium) et Observium sont des plateformes de monitoring réseau en PHP qui découvrent et interrogent les équipements en grande partie via SNMP, en stockant leurs identifiants pour ce faire. Les deux s'attaquent de la même manière. L'interface web s'authentifie avec un mot de passe et présente une exposition aux identifiants par défaut et faibles. Une fois authentifié, les deux ont connu des vulnérabilités d'injection de commandes dans des fonctionnalités qui construisent des commandes shell à partir d'entrées utilisateur, atteignant l'exécution de code sur l'hôte de monitoring. Et les deux stockent les chaînes de communauté SNMP et les identifiants d'équipements qu'ils utilisent pour interroger, de sorte que récolter ceux-ci donne accès à chaque équipement surveillé. La compromission de la plateforme se répercute donc en cascade sur le réseau qu'elle surveille.

```bash
curl -sk https://<target>/ | grep -iE 'librenms|observium'       # fingerprint
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification](authentication.md)** : identifiants par défaut et faibles.
- **[Injection de commandes](command-injection.md)** : les voies de RCE authentifiées.
- **[Récolte d'identifiants](credential-harvesting.md)** : chaînes SNMP et identifiants d'équipements stockés.

## Références

- [LibreNMS documentation](https://docs.librenms.org/)
- [LibreNMS security advisories](https://github.com/librenms/librenms/security/advisories)
