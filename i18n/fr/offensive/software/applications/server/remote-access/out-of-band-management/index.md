---
title: "Gestion hors bande : attaquer les BMC de serveurs sous le système d'exploitation"
order: 9
description: "La gestion hors bande donne le contrôle à distance du matériel d'un serveur indépendamment de son système d'exploitation, via le contrôleur de gestion de la carte mère (BMC) parlant IPMI et les piles des constructeurs (iDRAC, iLO) avec l'API Redfish. La surface est dévastatrice : le cipher 0 et les failles RAKP d'IPMI, des identifiants par défaut quasi universels et des exploits constructeur accordent un contrôle de l'alimentation, de la console et du média virtuel qui compromet le système d'exploitation de l'hôte."
keywords:
  - gestion hors bande
  - bmc
  - ipmi
  - idrac
  - ilo
---

# Gestion hors bande

La gestion hors bande contrôle le matériel d'un serveur à distance et indépendamment de son système d'exploitation, via un contrôleur de gestion de la carte mère (BMC) : un petit ordinateur toujours allumé sur la carte mère, doté de sa propre interface réseau, atteignable que le système d'exploitation de l'hôte soit en cours d'exécution ou non. Les BMC parlent IPMI et les piles des constructeurs (Dell iDRAC, HPE iLO) et exposent l'API REST Redfish. La sécurité y est notoirement médiocre, et l'impact est total : un BMC contrôle l'alimentation de l'hôte, la console physique (KVM) et le média virtuel (montage d'une image ISO malveillante), de sorte que le compromettre compromet l'hôte lui-même, en dessous et autour de toute défense du système d'exploitation. La surface se compose des failles de protocole d'IPMI, des identifiants par défaut omniprésents et des exploits des contrôleurs constructeur.

```bash
nmap -sU -p623 --script ipmi-version,ipmi-cipher-zero <target>   # IPMI/BMC on UDP 623
curl -sk https://<bmc>/redfish/v1/                               # Redfish API
```

## Sous-thèmes

- **[Contournement IPMI cipher 0](ipmi-cipher-0-bypass.md)** : la suite de chiffrement zéro qui désactive l'authentification.
- **[Divulgation de hachage RAKP IPMI](ipmi-rakp-hash-disclosure.md)** : récupération de hachage de mot de passe pré-authentification.
- **[Identifiants par défaut des BMC](bmc-default-credentials.md)** : identifiants administrateur livrés d'usine.
- **[iDRAC et iLO](idrac-and-ilo.md)** : exploits des contrôleurs Dell et HPE et média virtuel.
- **[API Redfish](redfish-api.md)** : détourner l'API REST moderne des BMC.

## Références

- [IPMI 2.0 specification](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
- [DMTF Redfish](https://www.dmtf.org/standards/redfish)
- [HackTricks: IPMI (623)](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
