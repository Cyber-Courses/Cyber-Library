---
title: "Détection de service : identifier le logiciel de bureau à distance par le port et le trafic"
description: "Chaque produit de bureau à distance utilise des ports et des points de terminaison de relais caractéristiques, TeamViewer sur 5938 (puis 443/80), AnyDesk sur 7070, Splashtop sur ses propres ports, de sorte que le balayage et l'analyse du trafic identifient lequel est en cours d'exécution. Sur un hôte, les processus, les fichiers installés et les clés de registre confirment le produit et pointent vers sa configuration et ses identifiants stockés."
keywords:
  - détection de service
  - port 5938
  - port 7070
  - relais
  - artefacts clients
---

# Détection de service

Identifier quel produit de bureau à distance est présent se fait sous deux angles. Sur le réseau, les clients utilisent des ports caractéristiques, TeamViewer privilégie TCP 5938 et se replie sur 443/80, AnyDesk utilise 7070 (et relaie sur 443), Splashtop et d'autres utilisent les leurs, de sorte qu'un balayage ciblé associé à l'observation des connexions vers l'infrastructure de relais de l'éditeur révèle le produit. Sur un hôte que vous pouvez inspecter, les processus en cours d'exécution, les fichiers de programme installés et les entrées de registre ou de données d'application nomment le produit avec précision et pointent vers l'emplacement de sa configuration et des éventuels identifiants stockés, ce qui compte pour la récupération des mots de passe d'accès sans surveillance.

```bash
# network detection
nmap -p5938,7070,6568,443,80 -sV <target>
# connections to vendor relays indicate the product even when local ports are filtered
# host artefacts (Windows)
tasklist | findstr /i 'teamviewer anydesk splashtop logmein'
dir "%APPDATA%\AnyDesk" "%ProgramData%\TeamViewer" 2>nul
```

## Notes d'exploitation

- Les ports caractéristiques et les destinations de relais identifient le produit même derrière un NAT, car le client se connecte vers l'extérieur au relais de l'éditeur ; surveillez ces connexions là où les ports entrants sont filtrés.
- Sur un hôte, l'emplacement de données d'application ou de registre du produit est aussi celui où sa configuration et ses identifiants sans surveillance sont stockés, de sorte que la détection de service alimente la récupération d'identifiants de l'[accès sans surveillance](../authentication/unattended-access.md).
- Plusieurs outils sont souvent installés (un outil géré plus un installé par l'utilisateur) ; énumérez-les tous, car une installation oubliée ou fantôme peut être la plus faible.
- Le produit identifié ainsi que sa [version](version-detection.md) sélectionnent l'attaque sur l'authentification et l'[exploit produit](../known-product-exploits/index.md) applicable.

## Références

- [TeamViewer / AnyDesk port documentation](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
