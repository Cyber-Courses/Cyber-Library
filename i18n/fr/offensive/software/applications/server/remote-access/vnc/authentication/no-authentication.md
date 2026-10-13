---
title: "Aucune authentification : serveurs VNC ne requérant aucun mot de passe"
description: "Un serveur VNC proposant le type de sécurité None (type 1) accorde un bureau interactif à quiconque se connecte, sans mot de passe. C'est une mauvaise configuration courante sur les outils de helpdesk, les équipements KVM-over-IP, les systèmes industriels et les VM, et elle se trouve simplement en se connectant ou en lisant les types de sécurité proposés pendant l'énumération."
keywords:
  - aucune authentification
  - type de sécurité none
  - vnc ouvert
  - non authentifié
  - accès au bureau
---

# Aucune authentification

La faiblesse VNC la plus simple est un serveur ne requérant aucune authentification du tout, proposant le type de sécurité RFB 1 (« None »). Quiconque se connecte obtient le bureau interactif. C'est répandu sur les systèmes configurés par commodité : outils de helpdesk et de partage d'écran, équipements KVM-over-IP et consoles out-of-band, HMI industriels, kiosques, et VM provisionnées rapidement. Cela ne nécessite aucune attaque, seulement l'accessibilité, et les scans à l'échelle d'internet cataloguent continuellement des bureaux VNC ouverts.

```bash
# the offered security type reveals it; type 1 = None
nmap -p5900-5905 --script vnc-info <target>    # look for "security types: None (1)"
# just connect; no password is requested
vncviewer <target>::5900
# search-engine discovery of open VNC (external): Shodan "port:5900 authentication disabled"
```

## Notes d'exploitation

- L'énumération est le détecteur : une liste de types de sécurité contenant `None (1)` signifie un accès au bureau non authentifié ; souvent c'est le seul type proposé.
- Ce sont fréquemment des consoles de grande valeur, une carte KVM-over-IP contrôle le matériel et le BIOS d'un serveur, un HMI contrôle un équipement industriel, un outil de helpdesk est connecté en tant qu'utilisateur, de sorte que « aucune auth » peut signifier bien plus qu'un seul bureau.
- Scannez la plage d'affichages (5900-590x) ; les hôtes multi-affichages et multi-VM exposent plusieurs points de terminaison VNC, pas tous configurés de la même façon.
- Un bureau ouvert est un contrôle interactif immédiat ; utilisez-le pour lire l'écran, exécuter des commandes dans toute session ouverte, et pivoter depuis le contexte de l'utilisateur connecté.

## Références

- [RFC 6143: None security type](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.1)
- [HackTricks: open VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
