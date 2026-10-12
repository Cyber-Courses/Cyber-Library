---
title: "Logiciels de bureau à distance : attaquer les outils tiers de bureau à distance"
order: 1
description: "Les produits tiers de bureau à distance, TeamViewer, AnyDesk, Chrome Remote Desktop, Splashtop et consorts, offrent un contrôle à distance en dehors des RDP/VNC natifs du système d'exploitation. La surface se compose de leur modèle d'ID-et-mot-de-passe ou d'accès sans surveillance, des mots de passe faibles et par défaut, de l'exposition sur Internet et via le relais, des réglages par défaut non sécurisés, et des failles de contournement d'authentification et d'exécution de code propres au produit."
keywords:
  - teamviewer
  - anydesk
  - logiciel de bureau à distance
  - accès sans surveillance
  - tiers
---

# Logiciels de bureau à distance

Aux côtés des protocoles d'accès à distance natifs du système d'exploitation, les organisations et les particuliers font tourner des produits tiers de bureau à distance, TeamViewer, AnyDesk, Chrome Remote Desktop, Splashtop, LogMeIn et d'autres, qui offrent un contrôle à distance via le client propre de l'éditeur, souvent par un relais cloud qui fonctionne à travers le NAT et les pare-feux. Cette conception déplace la surface d'attaque : l'accès repose sur un ID d'appareil plus un mot de passe (ou un mot de passe d'accès sans surveillance), les clients écoutent localement et se connectent vers des relais, et les produits ont leurs propres vulnérabilités d'authentification et d'exécution de code. La surface se compose du modèle d'ID/mot de passe et d'accès sans surveillance, des mots de passe faibles et par défaut, de l'exposition sur Internet et via le relais, des réglages par défaut non sécurisés, et des exploits produit nommés.

```bash
# detect common clients by their characteristic local/relay ports
nmap -p5938,7070,443 -sV <target>              # TeamViewer 5938, AnyDesk 7070, relays on 443
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : détecter le produit et la version.
- **[Authentification](authentication/index.md)** : accès par ID, accès sans surveillance et mots de passe faibles.
- **[Exposition](exposure/index.md)** : exposition sur Internet et réglages par défaut non sécurisés.
- **[Exploits produit connus](known-product-exploits/index.md)** : vulnérabilités de TeamViewer et AnyDesk.

## Références

- [TeamViewer security](https://www.teamviewer.com/en/trust-center/security/)
- [AnyDesk security](https://anydesk.com/en/security)
- [HackTricks: remote management software](https://book.hacktricks.xyz/)
