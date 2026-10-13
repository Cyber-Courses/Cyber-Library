---
title: "Transmission en clair : lire la session VNC non chiffrée"
description: "Le VNC standard envoie le framebuffer, les événements de saisie et le challenge-réponse d'authentification sans chiffrement. Un attaquant bien positionné capture le trafic pour reconstituer l'écran distant, récupérer les frappes clavier y compris les mots de passe saisis, et extraire le couple challenge-réponse en vue d'un cassage hors ligne du mot de passe VNC."
keywords:
  - clair
  - capture du framebuffer
  - capture des frappes
  - challenge réponse
  - sniffing
---

# Transmission en clair

Sans type de sécurité chiffrant (VeNCrypt/TLS), une session VNC est entièrement en clair sur le réseau. Pour un attaquant disposant d'une position de capture, cela donne trois choses : les mises à jour du framebuffer, qui reconstituent l'image du bureau distant que voit l'utilisateur ; les événements de saisie du client, qui incluent chaque frappe, de sorte que les mots de passe saisis dans la session sont capturés ; et la négociation d'authentification, dont le challenge et la réponse peuvent être prélevés pour un cassage hors ligne du mot de passe VNC court. Une seule session capturée peut ainsi donner à la fois le contenu et l'information d'identification.

```bash
# capture VNC traffic from an on-path position
tcpdump -i eth0 -w vnc.pcap port 5900
# reconstruct the screen from framebuffer updates, and read input events, in analysis
#   (tools and Wireshark dissectors parse RFB framebuffer/keyevent messages)
# extract the auth challenge+response for offline cracking (see password brute force)
```

## Notes d'exploitation

- Trois gains distincts à partir d'une seule capture : reconstruction de l'écran (ce que fait l'utilisateur), récupération des frappes (mots de passe et commandes qu'il saisit), et le couple challenge-réponse pour le [cassage hors ligne du mot de passe](../authentication/password-brute-force.md).
- Capturer les frappes donne souvent des informations d'identification pour d'autres systèmes que l'utilisateur saisit dans la session VNC, au-delà du mot de passe VNC lui-même.
- L'attaque nécessite une position sur le chemin (ARP/DNS/route) ou une prise réseau ; le VNC sur un LAN à plat est facilement intercepté.
- La défense consiste à tunneliser VNC (SSH/VPN) ou à utiliser VeNCrypt/TLS ; l'absence d'un type de sécurité chiffrant dans l'énumération confirme que la session est exposée.

## Références

- [RFC 6143 : absence de chiffrement par défaut](https://datatracker.ietf.org/doc/html/rfc6143)
- [HackTricks: VNC sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
