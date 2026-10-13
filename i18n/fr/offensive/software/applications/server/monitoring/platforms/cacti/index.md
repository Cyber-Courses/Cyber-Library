---
title: "Cacti : attaquer la plateforme de graphage et de monitoring"
order: 4
description: "Cacti est une application PHP de graphage et de monitoring réseau. Elle a un historique récurrent de vulnérabilités graves, injection de commandes et injection SQL non authentifiées et authentifiées atteignant l'exécution de code à distance, et elle stocke les chaînes SNMP et les identifiants d'équipements qu'elle utilise pour interroger. Les identifiants par défaut, les chaînes de RCE connues et la récolte d'identifiants constituent la surface."
keywords:
  - cacti
  - php
  - command injection
  - sql injection
  - snmp
---

# Cacti

Cacti est une application PHP open source pour interroger des équipements (en grande partie via SNMP) et graphiquer les résultats, couramment déployée sous `/cacti/`. Elle importe offensivement pour deux raisons : elle a un historique persistant de vulnérabilités web graves, injection de commandes et injection SQL non authentifiées et authentifiées dans ses composants de collecte de données et de graphage, plusieurs atteignant l'exécution de code à distance sur le serveur, et elle stocke les identifiants qu'elle utilise pour interroger les équipements (chaînes de communauté SNMP, et identifiants SSH/autres pour les sources de données), de sorte que la compromettre récolte les identifiants du parc. La surface est la connexion (par défaut `admin`/`admin`), les chaînes d'exploitation connues et la récolte d'identifiants stockés.

```bash
curl -sk https://<target>/cacti/ | grep -ioE 'Version [0-9.]+'   # fingerprint
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification](authentication.md)** : l'admin par défaut et les identifiants faibles.
- **[Exploits connus](known-exploits.md)** : les chaînes de RCE par SQLi et injection de commandes.
- **[Récolte d'identifiants](credential-harvesting.md)** : chaînes SNMP et identifiants d'équipements stockés.

## Références

- [Cacti documentation](https://docs.cacti.net/)
- [Cacti security advisories](https://github.com/Cacti/cacti/security/advisories)
