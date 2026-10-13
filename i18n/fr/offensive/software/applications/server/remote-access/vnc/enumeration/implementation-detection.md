---
title: "Détection d'implémentation : identifier le produit serveur VNC"
description: "Les implémentations VNC, RealVNC, TightVNC, UltraVNC, TigerVNC et les serveurs de plateforme, diffèrent par les types de sécurité pris en charge, les extensions et les chaînes de version, et chacune présente des vulnérabilités distinctes. Identifier l'implémentation à partir des types de sécurité proposés et du comportement relie la cible à ses failles connues de contournement d'authentification et de corruption mémoire."
keywords:
  - realvnc
  - tightvnc
  - ultravnc
  - fingerprint
  - type de sécurité
---

# Détection d'implémentation

Le protocole RFB est standard, mais les implémentations divergent dans les types de sécurité et les extensions qu'elles proposent ainsi que dans leurs chaînes de version, et ces différences sont porteuses de vulnérabilités. RealVNC, TightVNC, UltraVNC, TigerVNC et les serveurs macOS/embarqués prennent chacun en charge des schémas d'authentification différents (le type de sécurité Tight et ses sous-types, VeNCrypt, les schémas propriétaires) et ont leur propre historique de bugs, notamment des failles de contournement d'authentification dans RealVNC et TightVNC. Identifier l'implémentation restreint l'attaque aux problèmes connus de ce produit.

```bash
# the offered security types and extensions fingerprint the implementation
nmap -p5900 --script vnc-info <target>         # security types hint at Tight/UltraVNC/VeNCrypt
# behaviour and version strings distinguish products; vendor security types:
#   16 = Tight (TightVNC/TurboVNC), 17 = Ultra, 19 = VeNCrypt, 2 = standard VNC auth
# a connecting client that negotiates Tight sub-auth reveals TightVNC-family servers
```

## Notes d'exploitation

- Reliez l'implémentation à ses failles : RealVNC présente un contournement d'authentification connu lié à la gestion du type de sécurité, et TightVNC a connu des problèmes d'authentification et de sûreté mémoire ; identifier le produit sélectionne quel [contournement](../authentication/index.md) tenter.
- Les types de sécurité proposés sont le signal principal : les types Tight (16) et Ultra (17) indiquent ces familles, VeNCrypt (19) indique du VNC encapsulé dans TLS, et la seule présence du type standard 2 suggère un serveur basique.
- Le VNC embarqué et des appliances (sur l'IoT, le KVM-over-IP, les appareils industriels) exécute souvent d'anciens builds vulnérables avec des mots de passe par défaut ou inexistants ; l'implémentation et la version les signalent.
- Associez à la [version RFB](banner-grabbing.md) pour compléter le fingerprint avant d'attaquer.

## Références

- [Registre des types de sécurité RFB](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC implementations](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
