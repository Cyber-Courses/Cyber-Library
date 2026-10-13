---
title: "Récolte d'identifiants : identifiants d'équipements stockés dans LibreNMS/Observium"
description: "LibreNMS et Observium stockent les chaînes de communauté SNMP et les utilisateurs SNMPv3 qu'ils utilisent pour interroger chaque équipement supervisé, dans leurs bases de données et leur configuration. Avec un accès admin ou l'exécution de code, un attaquant les lit pour obtenir des identifiants valides pour tout le réseau supervisé, faisant de la plateforme un point unique depuis lequel compromettre les équipements qu'elle surveille."
keywords:
  - librenms credentials
  - snmp community
  - device credentials
  - database
  - harvesting
---

# Récolte d'identifiants

Pour interroger les équipements, les deux plateformes stockent les identifiants pour les atteindre, principalement les chaînes de communauté SNMP et les paramètres d'authentification et de confidentialité SNMPv3, par équipement dans leurs bases de données (la table `devices` dans LibreNMS, l'équivalent dans Observium) et dans la configuration. Un attaquant avec un accès admin les lit via l'UI (les paramètres SNMP de chaque équipement) et l'API, et avec l'exécution de code ou un accès à la base de données les lit en masse directement. Les chaînes récoltées sont immédiatement réutilisables contre les équipements, en lecture SNMP et souvent en lecture-écriture, et tout autre identifiant de collecteur stocké permet d'aller plus loin. Parce que ces plateformes supervisent tout le réseau, leur magasin d'identifiants est en pratique une clé maîtresse des équipements qu'elles surveillent.

```bash
# admin UI/API: per-device SNMP settings are shown and returned
curl -sk -H 'X-Auth-Token: <token>' https://<target>/api/v0/devices | jq '.devices[] | {hostname, community}'
# with DB access (via RCE or SQLi): read the device credentials in bulk
#   LibreNMS:  SELECT hostname, community, snmpver, authname, authpass, authalgo,
#              cryptopass, cryptoalgo FROM devices;
```

## Notes d'exploitation

- La table `devices` détient la communauté SNMP et les identifiants v3 de chaque hôte supervisé ; la lire procure un accès valide à tout le réseau supervisé pour l'[énumération et l'écriture SNMP](../../protocols/snmp/index.md).
- L'accès à la base de données (via [l'injection de commandes](command-injection.md) ou une SQLi) est la récolte en masse la plus propre ; l'API et l'UI exposent les mêmes données par équipement.
- Les chaînes de communauté en lecture-écriture récoltées permettent la reconfiguration des équipements (et l'exfiltration de configuration Cisco) ; les identifiants v3 et tout autre login de collecteur permettent de pivoter vers ces services.
- Cela rend la plateforme précieuse même sans RCE ultérieure : ses identifiants stockés sont un accès au réseau qu'elle supervise.

## Références

- [LibreNMS: device/SNMP configuration](https://docs.librenms.org/)
- [Observium documentation](https://docs.observium.org/)
