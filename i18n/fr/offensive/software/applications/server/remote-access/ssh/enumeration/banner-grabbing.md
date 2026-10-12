---
title: "Banner grabbing : divulgation de la version et de l'implémentation SSH"
description: "Un serveur SSH annonce sa version de protocole et son logiciel dans une bannière en clair avant l'authentification, par exemple SSH-2.0-OpenSSH_8.9p1 Ubuntu. Cette chaîne identifie l'implémentation, la version et souvent l'OS/la distribution, reliant directement le service à ses vulnérabilités connues et orientant l'énumération des utilisateurs et le choix des exploits."
keywords:
  - bannière ssh
  - détection de version
  - version openssh
  - empreinte
  - identification
---

# Banner grabbing

La première chose qu'envoie un serveur SSH, avant toute authentification et en clair, est sa bannière d'identification : une ligne comme `SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4`. Cette unique chaîne donne la version du protocole, l'implémentation (OpenSSH, Dropbear, libssh, une pile constructeur), la version exacte du logiciel, et souvent l'OS ou le conditionnement de la distribution. C'est le point d'ancrage de tout le reste : la version renvoie aux vulnérabilités connues de l'implémentation, distingue OpenSSH de Dropbear (courant sur les équipements embarqués, avec ses propres bogues), et vous indique quelles techniques d'énumération des utilisateurs et d'exploitation s'appliquent à cette version.

```bash
# grab the banner several ways
nc <target> 22                                 # prints SSH-2.0-...
echo | nc <target> 22 | head -1
nmap -p22 -sV <target>                         # service/version with OS hints
# at scale across a range
for h in $(cat hosts.txt); do echo "$h $(echo | nc -w2 $h 22 | head -1)"; done
```

## Notes d'exploitation

- L'implémentation et la version sont les faits clés : OpenSSH, Dropbear, libssh et les forks constructeurs ont des historiques de vulnérabilités distincts, et la version exacte renvoie à des avis de sécurité précis (failles d'énumération des utilisateurs, contournements d'authentification, bogues de corruption mémoire).
- Les chaînes de distribution (suffixes de backport Ubuntu, Debian) affinent le niveau de correctifs, puisque les distributions rétroportent les correctifs sans changer le numéro de version amont.
- Les bannières Dropbear indiquent un équipement embarqué/IoT, orientant vers les [identifiants par défaut](../authentication/default-credentials.md) et les problèmes propres à l'équipement.
- La bannière est en clair et pré-authentification, donc cela ne coûte rien et constitue la première étape ; injectez la version dans une recherche de vulnérabilités et l'implémentation dans la bonne méthode d'énumération des utilisateurs.

## Références

- [HackTricks : bannière SSH](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [Notes de version d'OpenSSH (correspondance des versions)](https://www.openssh.com/releasenotes.html)
