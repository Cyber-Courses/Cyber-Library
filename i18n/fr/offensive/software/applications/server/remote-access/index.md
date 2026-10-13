---
title: "Accès à distance : attaquer les services d'administration à distance"
order: 8
description: "Les services d'accès à distance permettent aux administrateurs et aux utilisateurs de contrôler des systèmes sur le réseau, et ils constituent une cible de premier plan parce qu'ils ouvrent directement sur un shell, un bureau ou un réseau interne. Le domaine couvre SSH, RDP, VNC, Telnet, les r-commandes de Berkeley, les passerelles VPN, WinRM, la gestion BMC hors bande et les logiciels tiers de bureau à distance, classés par service puis par aspect."
keywords:
  - accès à distance
  - ssh
  - rdp
  - vpn
  - administration à distance
---

# Accès à distance

Les services d'accès à distance existent pour contrôler un système depuis ailleurs, de sorte qu'en compromettre un est, par définition, une voie directe vers un shell, un bureau ou le réseau interne derrière une passerelle. Cela les place parmi les cibles les plus précieuses d'un réseau et en fait le principal vecteur d'accès initial en pratique. Les services diffèrent par le protocole et l'époque mais partagent un ensemble récurrent d'aspects : énumération et relevé d'empreinte, authentification (identifiants, clés, confiance), cryptographie du transport, failles de protocole ou d'implémentation, et détournement de l'accès une fois obtenu. Le domaine est organisé par service, avec ces aspects sous chacun.

## Trouver les services d'accès à distance

```bash
# one sweep across the common remote-access ports
nmap -sV -p22,23,512,513,514,3389,5900-5902,5985,5986,443,500,1723,623 <target>
nmap -sU -p500,4500,623 <target>               # IKE/IPsec and IPMI (UDP)
#  22 SSH  23 Telnet  512-514 r-cmds  3389 RDP  5900+ VNC  5985/6 WinRM
#  443 SSL-VPN/portal  500/4500 IPsec  1723 PPTP  623 IPMI/BMC
```

## Sous-thèmes

- **[SSH](ssh/index.md)** : le shell distant chiffré, les clés, la cryptographie, la confiance et le tunnelage.
- **[RDP](rdp/index.md)** : le bureau à distance de Windows, NLA, RCE pré-authentification et détournement de session.
- **[VNC](vnc/index.md)** : le bureau RFB, authentification faible et clair.
- **[Telnet](telnet/index.md)** : le terminal en clair et les failles de telnetd.
- **[Protocoles shell](shell-protocols/index.md)** : les r-commandes de Berkeley et la confiance entre hôtes.
- **[VPN](vpn/index.md)** : passerelles d'accès à distance, protocoles et exploits des équipements SSL-VPN.
- **[WinRM](winrm/index.md)** : Windows Remote Management et PowerShell Remoting.
- **[Gestion hors bande](out-of-band-management/index.md)** : BMC, IPMI, iDRAC/iLO et Redfish.
- **[Logiciels de bureau à distance](desktop-software/index.md)** : outils tiers de bureau à distance.

## Références

- [CISA: remote-access exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [MITRE ATT&CK: remote services](https://attack.mitre.org/techniques/T1021/)
- [HackTricks: network services pentesting](https://book.hacktricks.xyz/network-services-pentesting)
