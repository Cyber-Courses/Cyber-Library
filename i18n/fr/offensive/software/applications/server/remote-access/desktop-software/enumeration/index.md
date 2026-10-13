---
title: "Énumération : détecter le logiciel de bureau à distance et sa version"
description: "Les outils de bureau à distance se détectent par les ports d'écoute locaux caractéristiques et le trafic de relais, TeamViewer sur 5938, AnyDesk sur 7070, et d'autres, ainsi que par les artefacts clients présents sur un hôte. Identifier le produit et sa version oriente l'attaque sur l'authentification et associe le client à ses exploits produit connus."
keywords:
  - détection de service
  - détection de version
  - port 5938
  - port 7070
  - empreinte
---

# Énumération

Énumérer ces outils permet d'identifier quel produit est présent et sa version, ce qui conditionne à la fois l'attaque sur l'authentification et le choix de l'exploit produit. La détection repose sur les ports caractéristiques utilisés par les clients, TeamViewer sur TCP 5938 (avec repli sur 443/80), AnyDesk sur 7070, Splashtop et d'autres sur les leurs, ainsi que sur le trafic de relais de l'éditeur, auxquels s'ajoutent les artefacts clients (processus, fichiers installés, registre) sur un hôte que vous pouvez inspecter. La détection de version importe ensuite parce que ces produits sont corrigés fréquemment et que de nombreuses attaques (faisabilité du brute-force, exploits spécifiques) dépendent de la version.

```bash
nmap -p5938,7070,6568,443 -sV <target>         # TeamViewer, AnyDesk, Splashtop, relay
# on a host: identify the client and version from processes/installed files
tasklist | findstr /i 'teamviewer anydesk splashtop'    # Windows
```

## Sous-thèmes

- **[Détection de service](service-detection.md)** : identifier le produit par le port et le trafic.
- **[Détection de version](version-detection.md)** : déterminer la version du client.

## Références

- [TeamViewer ports](https://www.teamviewer.com/en/)
- [AnyDesk ports](https://anydesk.com/en/)
