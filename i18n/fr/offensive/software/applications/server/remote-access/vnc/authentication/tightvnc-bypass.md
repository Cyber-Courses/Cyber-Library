---
title: "Contournement TightVNC : failles d'authentification et de sûreté mémoire"
description: "TightVNC et ses dérivés ont comporté des vulnérabilités de gestion de l'authentification et de corruption mémoire, notamment des failles dans la négociation de sous-authentification du type de sécurité Tight et dans la gestion des tampons, qui permettent de contourner la vérification du mot de passe ou de faire planter et exploiter le serveur. L'identification de la version de TightVNC relie une cible à la faille applicable."
keywords:
  - tightvnc
  - type de sécurité tight
  - contournement d'authentification
  - corruption mémoire
  - rfb
---

# Contournement TightVNC

TightVNC (et des dérivés comme TurboVNC et certains forks embarqués) introduit le type de sécurité Tight, qui ajoute une négociation de sous-authentification par-dessus la négociation RFB de base, et son code a comporté à la fois des failles de gestion de l'authentification et des bugs de sûreté mémoire. Les problèmes d'authentification permettent d'influencer ou de sauter l'étape de sous-authentification pour atteindre une session sans identifiants valides sur les versions affectées ; les problèmes de corruption mémoire (dans la gestion des messages et des encodages) permettent de faire planter ou d'exploiter le serveur. La bonne attaque dépend de la version exacte de TightVNC, de sorte que l'identification est la première étape.

```bash
# identify TightVNC and its version via the offered Tight security type and behaviour
nmap -p5900 --script vnc-info <target>         # Tight (16) security type indicates the family
# match the version to the applicable advisory: auth-handling bypass vs memory-corruption
# auth path: manipulate the Tight sub-auth negotiation to avoid the password check
# memory path: send crafted messages/encodings that the server mishandles
```

## Notes d'exploitation

- L'étape supplémentaire de sous-authentification du type de sécurité Tight constitue la surface distinctive ; des failles spécifiques à certaines versions y permettent d'atteindre une session sans le mot de passe, dans le même esprit que le contournement RealVNC mais dans la négociation Tight.
- Les bugs de corruption mémoire dans la gestion par TightVNC des messages et encodages RFB forment une classe distincte, donnant des plantages ou de l'exécution de code contre le serveur ; ils sont spécifiques au build.
- Les appareils embarqués et les appliances embarquent souvent d'anciens forks de TightVNC avec des versions non corrigées de ces failles, plus des mots de passe faibles ou par défaut ; identifiez l'[implémentation](../enumeration/implementation-detection.md) et la version.
- Là où aucun contournement ne s'applique, le schéma de mot de passe reste faible, voir [Force brute de mot de passe](password-brute-force.md) et [Mots de passe par défaut](default-passwords.md).

## Références

- [Avis de sécurité TightVNC](https://www.tightvnc.com/)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
