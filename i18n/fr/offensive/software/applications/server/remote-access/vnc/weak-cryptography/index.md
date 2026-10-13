---
order: 2
title: "Cryptographie faible : faiblesses du transport et du type de sécurité VNC"
description: "Le VNC standard ne chiffre pas la session, de sorte que le contenu de l'écran, les frappes clavier et le challenge-réponse d'authentification traversent le réseau en clair. La négociation du type de sécurité RFB peut aussi être dégradée vers un type plus faible ou sans authentification, et le chiffrement que certaines implémentations ajoutent est souvent faible ou optionnel, laissant le trafic VNC capturé et les informations d'identification récupérables."
keywords:
  - chiffrement vnc
  - clair
  - type de sécurité rfb
  - downgrade
  - cryptographie faible
---

# Cryptographie faible

Le VNC de base n'a pas de chiffrement de transport : la session RFB, les mises à jour du framebuffer (l'écran), la saisie du client (frappes clavier et souris), et même le challenge-réponse d'authentification circulent en clair. Un attaquant bien positionné lit donc le bureau et capture le matériel d'authentification. En plus de cela, le type de sécurité est négocié et peut être dégradé vers une option plus faible ou sans authentification, et le chiffrement que certaines implémentations greffent (schémas propriétaires, VeNCrypt/TLS) est souvent faible, optionnel ou non vérifié, de sorte qu'il échoue fréquemment à protéger la session en pratique.

```bash
# confirm the session is cleartext (no VeNCrypt/TLS security type)
nmap -p5900 --script vnc-info <target>         # security types; absence of VeNCrypt(19) => cleartext
tcpdump -i eth0 -w vnc.pcap port 5900          # capture for analysis
```

## Sous-thèmes

- **[Transmission en clair](cleartext-transmission.md)** : lire le bureau et les informations d'identification non chiffrés.
- **[Chiffrement faible](weak-encryption.md)** : modes de chiffrement faibles ou optionnels.
- **[Downgrade du type de sécurité](security-type-downgrade.md)** : forcer un type de sécurité plus faible.

## Références

- [RFC 6143 : types de sécurité](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
