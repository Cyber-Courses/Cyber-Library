---
title: "Chiffrement faible : modes de chiffrement optionnels et faibles de VNC"
description: "Là où les implémentations VNC ajoutent du chiffrement, il est souvent faible ou optionnel : schémas propriétaires à clés petites ou fixes, TLS obsolète dans les configurations VeNCrypt, et chiffrement que le serveur propose mais n'exige pas. Ces éléments laissent la session récupérable pour un attaquant qui capture, ou dégradable, en deçà de la protection que leur présence laisse supposer."
keywords:
  - chiffrement faible
  - venCrypt
  - chiffrement propriétaire
  - chiffrement optionnel
  - tls
---

# Chiffrement faible

Parce que le VNC de base est en clair, les implémentations ont ajouté du chiffrement, mais il est fréquemment insuffisant. Certains schémas propriétaires utilisent des clés petites ou fixes ou des algorithmes propriétaires à la robustesse douteuse. VeNCrypt encapsule VNC dans TLS, mais les déploiements utilisent des versions de TLS obsolètes, des chiffrements faibles, ou des certificats auto-signés non vérifiés, de sorte qu'un attaquant bien positionné intercepte malgré tout. Et surtout, le chiffrement est souvent proposé mais non exigé, de sorte qu'un client (ou un downgrade forcé) peut choisir un type en clair ou faible à la place. La présence d'une option de chiffrement ne signifie pas que la session est réellement protégée.

```bash
# what encryption/security types are offered, and are they required?
nmap -p5900 --script vnc-info <target>         # VeNCrypt(19)/vendor types present?
# VeNCrypt wraps TLS: assess it like any TLS endpoint
openssl s_client -connect <target>:5900 2>/dev/null | head   # (after the RFB/VeNCrypt negotiation)
# if encryption is optional, select/force a cleartext or weak type instead
```

## Notes d'exploitation

- La question clé est de savoir si le chiffrement est exigé ou simplement proposé ; un schéma optionnel se contourne en choisissant un type plus faible, voir [downgrade du type de sécurité](security-type-downgrade.md).
- VeNCrypt/TLS n'est robuste que dans la mesure de sa configuration et de sa validation de certificat ; des chiffrements faibles ou un certificat auto-signé non vérifié permettent à un attaquant bien positionné de faire du MITM, comme pour tout TLS faible.
- Un chiffrement propriétaire à clés fixes ou petites offre peu de protection réelle ; traitez-le comme de l'obfuscation et tentez la capture/le déchiffrement.
- Là où aucun chiffrement effectif n'est en vigueur, la session est aussi exposée qu'en [transmission en clair](cleartext-transmission.md).

## Références

- [VeNCrypt et types de sécurité RFB](https://datatracker.ietf.org/doc/html/rfc6143#section-7.1.2)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
