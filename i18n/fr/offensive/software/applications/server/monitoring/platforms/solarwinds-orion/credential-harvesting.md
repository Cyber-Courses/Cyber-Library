---
order: 2
title: "Récolte d'identifiants : identifiants d'équipements et réseau stockés dans Orion"
description: "Pour interroger et gérer les équipements, Orion stocke leurs identifiants, chaînes SNMP, connexions SSH et WMI, et secrets d'API, chiffrés dans sa base de données avec des clés situées sur le serveur Orion. Avec un accès admin ou une compromission du serveur/de la base de données, un attaquant récupère et déchiffre ces éléments pour obtenir des identifiants fonctionnels pour tout le parc géré, y compris les comptes de domaine qu'Orion utilise pour WMI."
keywords:
  - solarwinds credentials
  - orion database
  - snmp wmi
  - domain account
  - harvesting
---

# Récolte d'identifiants

Le rôle central d'Orion signifie qu'il stocke des identifiants pour tout ce qu'il gère : chaînes de communauté SNMP pour les équipements réseau, connexions SSH pour les systèmes réseau et Unix, identifiants Windows/WMI (fréquemment des comptes de domaine) pour les serveurs, et clés d'API pour les intégrations. Ceux-ci sont conservés dans la base de données SQL Server d'Orion, chiffrés avec des clés détenues sur le serveur Orion. Un attaquant qui atteint la base de données (via l'[exécution de code](known-exploits.md), une injection SQL ou un accès à la base) ainsi que le matériel de clés du serveur récupère et déchiffre ces éléments, obtenant des identifiants fonctionnels pour l'ensemble du parc géré. Les identifiants WMI/Windows sont particulièrement précieux car Orion s'exécute souvent avec des comptes de domaine privilégiés pour gérer les serveurs, de sorte que la récolte du magasin d'identifiants d'Orion inclut fréquemment un accès de domaine à privilèges élevés, en plus du SNMP (réutilisable pour lire/reconfigurer des équipements) et des connexions SSH.

```bash
# with DB access (via RCE/SQLi), the credentials live in the Orion database,
# encrypted with keys on the Orion server; recover both, then decrypt:
#   the Credentials/NodeSettings tables hold encrypted SNMP/SSH/WMI secrets
#   decryption uses the Orion server's key material (DPAPI/config keys)
# admin console access also exposes/edits node credentials per device
```

## Notes d'exploitation

- Le magasin d'identifiants d'Orion est une clé maîtresse du parc géré : les chaînes SNMP (réutilisables pour la lecture/écriture d'équipements), les connexions SSH et les identifiants WMI/Windows (souvent des comptes de domaine privilégiés) en sortent tous.
- Les identifiants WMI/de domaine ont la plus grande valeur, car Orion gère couramment des serveurs Windows avec des comptes privilégiés ; les récolter peut livrer un accès à l'échelle du domaine.
- Le déchiffrement nécessite le matériel de clés du serveur Orion en plus de la base de données, donc associez l'accès à la base avec un accès au serveur (exécution de code) pour la récupération complète.
- Cela rend Orion digne d'être compromis même sans RCE ultérieure ; ses identifiants stockés sont un accès au réseau et souvent au domaine, donc priorisez la récolte.

## Références

- [SolarWinds Orion: credential management](https://documentation.solarwinds.com/)
- [SolarWinds security advisories](https://www.solarwinds.com/trust-center/security-advisories)
