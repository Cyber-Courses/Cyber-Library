---
title: "Récupération de bannière : la version du protocole RFB de VNC"
description: "Un serveur VNC ouvre la négociation RFB en envoyant sa version de protocole sous la forme d'une chaîne fixe de 12 octets, par exemple RFB 003.008, avant toute authentification. Cette version, et les types de sécurité que le serveur propose ensuite, identifient la génération du protocole et l'authentification que le serveur acceptera, orientant le choix de l'attaque."
keywords:
  - version rfb
  - bannière vnc
  - handshake
  - types de sécurité
  - 5900
---

# Récupération de bannière

Le protocole RFB commence par l'envoi, par le serveur, d'une chaîne de version de 12 octets comme `RFB 003.008\n`, immédiatement à la connexion et avant toute authentification. Sa lecture identifie la génération du protocole (3.3, 3.7, 3.8), ce qui importe car les versions plus anciennes négocient la sécurité différemment (en 3.3 le serveur dicte l'unique type de sécurité ; 3.7+ proposent une liste parmi laquelle le client choisit, ce qui permet le downgrade). Après la version, le serveur présente les types de sécurité qu'il accepte, la première indication réelle de la nécessité ou non d'une authentification.

```bash
nc <target> 5900 | head -c 12                  # "RFB 003.008"
nmap -p5900 --script vnc-info <target>         # version + security type list
# the security-type byte(s) that follow: 1 = None, 2 = VNC auth (weak), 16/18 = Tight/VeNCrypt
```

## Notes d'exploitation

- La chaîne de version est pré-authentification et gratuite ; `003.003` par rapport à `003.007`/`003.008` vous dit si le client peut choisir le type de sécurité (permettant le [downgrade du type de sécurité](../weak-cryptography/security-type-downgrade.md)) ou si le serveur le dicte.
- Les types de sécurité proposés juste après sont le fait décisif : le type de sécurité 1 est « None » ([pas d'authentification](../authentication/no-authentication.md)), le type 2 est le mot de passe VNC DES faible, et les types propres aux éditeurs (Tight, VeNCrypt) pointent vers une gestion spécifique à l'implémentation.
- Combinez la version avec la [détection d'implémentation](implementation-detection.md) pour relier aux contournements connus (RealVNC/TightVNC).
- Les affichages au-delà du premier se trouvent sur 5901, 5902, etc. ; scannez la plage, car il est courant d'avoir plusieurs serveurs VNC sur un même hôte.

## Références

- [RFC 6143 : handshake](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
