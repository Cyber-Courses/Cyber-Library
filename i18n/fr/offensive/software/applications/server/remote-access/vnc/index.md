---
order: 6
title: "VNC : attaquer Virtual Network Computing"
description: "VNC partage un bureau graphique via le protocole RFB (généralement les ports 5900+). Ses faiblesses sont un schéma de mot de passe challenge-réponse court plafonné à huit caractères, des serveurs laissés sans authentification, un trafic d'écran et de saisie en clair, un type de sécurité négociable pouvant être dégradé, et des contournements d'authentification spécifiques à l'implémentation dans RealVNC et TightVNC."
keywords:
  - vnc
  - protocole rfb
  - port 5900
  - challenge des
  - bureau à distance
---

# VNC

VNC (Virtual Network Computing) partage un bureau graphique via le protocole Remote Framebuffer (RFB), habituellement sur TCP 5900 et au-dessus (l'affichage N sur 5900+N). Sa sécurité est faible par conception et par déploiement. L'authentification VNC classique est un challenge-réponse DES utilisant un mot de passe tronqué à huit caractères, trivial à brute forcer ou à casser à partir d'un challenge capturé. De nombreux serveurs fonctionnent sans aucune authentification. Le contenu de l'écran et les frappes clavier circulent en clair à moins d'être tunnelisés. Le type de sécurité RFB est négocié et peut être dégradé. Et des implémentations spécifiques (RealVNC, TightVNC) ont livré des contournements d'authentification purs et simples.

```bash
nmap -p5900-5902 --script vnc-info,vnc-title,realvnc-auth-bypass <target>
vncviewer <target>::5900                       # connect to test auth/no-auth
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : version RFB, types de sécurité et implémentation.
- **[Authentification](authentication/index.md)** : pas d'authentification, mots de passe par défaut, brute force et contournements.
- **[Cryptographie faible](weak-cryptography/index.md)** : clair, chiffrement faible et downgrade.

## Références

- [RFC 6143 (protocole RFB)](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC (5900)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
