---
title: "RDP : attaquer le Remote Desktop Protocol"
order: 2
description: "RDP sur TCP 3389 est le protocole d'administration à distance graphique de Windows et l'un des services exposés sur internet les plus attaqués. La surface couvre l'énumération de la version et des paramètres de sécurité, les attaques d'authentification et les considérations liées à la NLA, l'exposition sur internet et un TLS faible, la RCE de protocole en pré-authentification, et l'abus de sessions : détournement, shadowing et redirection de périphériques."
keywords:
  - rdp
  - remote desktop protocol
  - port 3389
  - nla
  - terminal services
---

# RDP

RDP (Remote Desktop Protocol) est le service d'administration à distance graphique de Windows, à l'écoute sur TCP 3389, et il figure parmi les services les plus lourdement attaqués sur internet : un RDP exposé est un vecteur d'accès initial primordial pour les rançongiciels. Sa surface couvre l'énumération (version, paramètres de chiffrement et de NLA, certificat TLS), l'authentification (force brute, spraying, credential stuffing, et le rôle de l'authentification au niveau du réseau), l'exposition (points d'accès exposés sur internet et TLS faible), les vulnérabilités de protocole en pré-authentification (la classe de RCE propageable comme un ver), et l'abus de sessions : prise de contrôle, shadowing, ou utilisation des canaux de redirection de périphériques des sessions RDP.

```bash
nmap -p3389 -sV --script rdp-ntlm-info,rdp-enum-encryption <target>
# NLA/security posture and NTLM-leaked host/domain info
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : version, paramètres de sécurité et certificat.
- **[Authentification](authentication/index.md)** : force brute, spraying, stuffing et NLA.
- **[Exposition](exposure/index.md)** : RDP exposé sur internet et TLS faible.
- **[Failles de pré-authentification](pre-authentication-flaws/index.md)** : RCE de protocole propageable comme un ver.
- **[Abus de session](session-abuse/index.md)** : détournement, shadowing et redirection de périphériques.

## Références

- [MS-RDPBCGR protocol specification](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/)
- [HackTricks: RDP (3389)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
