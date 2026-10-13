---
order: 3
title: "Énumération : fingerprinting d'un service VNC"
description: "L'énumération VNC lit la version du protocole RFB et les types de sécurité que le serveur propose pendant la négociation non authentifiée, et identifie l'implémentation (RealVNC, TightVNC, UltraVNC). Les types de sécurité révèlent si le serveur n'exige aucune authentification ou le schéma de mot de passe VNC faible, et l'implémentation se relie à des failles spécifiques de contournement d'authentification."
keywords:
  - énumération vnc
  - version rfb
  - types de sécurité
  - implémentation
  - vnc-info
---

# Énumération

La négociation de VNC est informative avant toute authentification : le serveur annonce sa version du protocole RFB et la liste des types de sécurité qu'il prend en charge, et l'implémentation peut être identifiée à partir des chaînes de version et du comportement. La liste des types de sécurité est le constat clé, elle montre si le serveur propose « None » (pas d'authentification), le schéma de mot de passe VNC DES faible, ou des schémas propriétaires, et l'implémentation (RealVNC, TightVNC, UltraVNC) se relie à des vulnérabilités connues de contournement d'authentification, de sorte que l'énumération oriente toute l'attaque.

```bash
nmap -p5900 --script vnc-info <target>         # RFB version + supported security types
# raw handshake: the server sends "RFB 003.00X" then the security-type list
nc <target> 5900 | head -c 12                  # RFB protocol version banner
```

## Sous-thèmes

- **[Récupération de bannière](banner-grabbing.md)** : la chaîne de version RFB.
- **[Détection d'implémentation](implementation-detection.md)** : identifier RealVNC, TightVNC, UltraVNC.

## Références

- [RFC 6143 : version de protocole et types de sécurité](https://datatracker.ietf.org/doc/html/rfc6143)
- [nmap vnc-info](https://nmap.org/nsedoc/scripts/vnc-info.html)
