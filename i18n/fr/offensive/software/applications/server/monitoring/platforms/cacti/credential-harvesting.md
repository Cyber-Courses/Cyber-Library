---
order: 3
title: "Récolte d'identifiants : chaînes SNMP et identifiants d'équipements stockés"
description: "Cacti stocke les identifiants qu'il utilise pour interroger les équipements, les chaînes de communauté SNMP et les utilisateurs SNMPv3, ainsi que les identifiants d'autres collecteurs de données, dans sa base de données et ses paramètres. Avec un accès admin ou l'exécution de code, un attaquant les lit pour obtenir des identifiants valides pour chaque équipement supervisé, transformant une compromission de Cacti en un large accès sur tout le réseau qu'il surveille."
keywords:
  - identifiants Cacti
  - communauté SNMP
  - source de données
  - base de données
  - récolte
---

# Récolte d'identifiants

Cacti doit s'authentifier auprès des équipements qu'il interroge, donc il stocke leurs identifiants, et ceux-ci sont un butin de premier choix. Les chaînes de communauté SNMP et les paramètres des utilisateurs SNMPv3 sont stockés par équipement (et dans les gabarits d'hôtes) pour les interroger, et les sources de données pour d'autres méthodes de collecte détiennent leurs propres identifiants. Ceux-ci vivent dans la base de données MySQL de Cacti (par exemple la table `host` et les paramètres) et sont lisibles via l'interface admin et l'API, et directement depuis la base de données avec l'exécution de code ou une injection SQL. Les récolter procure des identifiants valides pour chaque équipement supervisé, donc une seule compromission de Cacti devient un accès sur tout le parc qu'il surveille, les chaînes SNMP réutilisables pour lire (et souvent écrire) les équipements, et tout login d'équipement stocké réutilisable directement.

```bash
# with admin UI access: device SNMP settings are shown per host in the console
#   console > Devices > <host> reveals the SNMP community/version/credentials
# with DB access (via RCE or SQLi), read them directly
#   SELECT description, hostname, snmp_community, snmp_version, snmp_username, snmp_password
#   FROM host;
# settings table holds additional stored secrets
```

## Notes d'exploitation

- La table `host` (et les gabarits d'hôtes) détiennent les chaînes de communauté SNMP et les identifiants SNMPv3 par équipement que Cacti utilise pour interroger ; ils sont immédiatement réutilisables contre les équipements pour l'[énumération et l'écriture SNMP](../../protocols/snmp/index.md).
- Atteindre la base de données (via [l'exécution de code](known-exploits.md) ou une SQLi) donne la récolte en masse la plus propre ; l'interface admin expose les mêmes données par équipement.
- Les chaînes SNMP récoltées incluent souvent des chaînes en lecture-écriture (pour le contrôle des interfaces/configurations), et tout autre identifiant de source de données stocké permet d'aller plus loin.
- Cela rend Cacti précieux même sans RCE ultérieure : les identifiants stockés sont un accès à tout le réseau supervisé.

## Références

- [Cacti: device and data-source configuration](https://docs.cacti.net/)
- [HackTricks: Cacti](https://book.hacktricks.xyz/)
