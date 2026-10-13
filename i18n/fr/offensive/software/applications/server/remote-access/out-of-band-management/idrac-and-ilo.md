---
title: "iDRAC et iLO : exploits des contrôleurs Dell et HPE et média virtuel"
order: 4
description: "Dell iDRAC et HPE iLO sont les piles BMC des constructeurs, et tous deux ont connu des failles graves : contournements d'authentification et exécution de code à distance atteignables sur l'interface de gestion. Au-delà des exploits, leurs fonctionnalités légitimes de média virtuel et de console permettent à un attaquant disposant d'un accès de monter une image ISO malveillante et de démarrer l'hôte dessus, compromettant directement le système d'exploitation."
keywords:
  - idrac
  - ilo
  - contournement d'authentification
  - média virtuel
  - exécution de code à distance
---

# iDRAC et iLO

Dell iDRAC et HPE iLO sont les contrôleurs de gestion des constructeurs bâtis sur le BMC, et ils s'attaquent de deux façons. Premièrement, les vulnérabilités produit : tous deux ont connu des failles de contournement d'authentification et d'exécution de code à distance atteignables sur l'interface web de gestion et les API (iLO a notamment eu un contournement d'authentification permettant un accès administrateur non authentifié, et iDRAC a connu une RCE non authentifiée), de sorte que relever l'empreinte du contrôleur et de la version du micrologiciel permet de le rapprocher de son exploit. Deuxièmement, et indépendamment de tout bug, leurs fonctionnalités légitimes sont elles-mêmes l'arme : avec un accès administratif (par défauts, hachages IPMI cassés ou exploit), la fonctionnalité de média virtuel monte une image ISO fournie par l'attaquant et le contrôleur démarre l'hôte dessus, donnant une compromission directe du système d'exploitation du serveur.

```bash
# fingerprint the controller and firmware
curl -sk https://<bmc>/ | grep -iE 'idrac|integrated lights-out|ilo'
curl -sk https://<bmc>/redfish/v1/ | grep -i 'Model\|FirmwareVersion'
# match firmware to the advisory for an auth-bypass/RCE (version-specific)
# with admin access: mount attacker media and boot the host from it
#   iDRAC/iLO virtual media -> attach ISO (SMB/HTTP/local) -> set one-time boot -> power cycle
```

## Notes d'exploitation

- Deux voies distinctes : un exploit produit (contournement d'authentification ou RCE, spécifique à la version, relevez l'empreinte du micrologiciel et rapprochez-la de l'avis) donne l'accès au contrôleur sans identifiants ; et le détournement légitime du média virtuel et de la console, une fois que vous avez un accès administrateur par un moyen quelconque, compromet le système d'exploitation de l'hôte.
- Le média virtuel est la fonctionnalité décisive : monter une image ISO malveillante et définir un démarrage unique permet au serveur de démarrer sur du code contrôlé par l'attaquant (un système d'exploitation en direct pour lire ou modifier le disque, réinitialiser l'administrateur local ou implanter), contournant toutes les défenses au niveau du système d'exploitation.
- La console (KVM) donne en outre un accès interactif au système d'exploitation en cours d'exécution et à son écran de connexion, utile avec d'autres attaques sur les identifiants.
- L'accès administrateur au contrôleur provient des [défauts](bmc-default-credentials.md), des [hachages RAKP cassés](ipmi-rakp-hash-disclosure.md), du [cipher 0](ipmi-cipher-0-bypass.md) ou d'un exploit ; l'un quelconque de ceux-ci plus le média virtuel équivaut à une compromission totale de l'hôte.

## Références

- [Dell iDRAC security advisories](https://www.dell.com/support/security/)
- [HPE iLO security advisories](https://support.hpe.com/)
