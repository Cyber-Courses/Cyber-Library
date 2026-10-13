---
title: "Sniffing de mot de passe : capturer les identifiants Telnet sur le fil"
description: "Le login Telnet est en clair, de sorte qu'un attaquant positionné capture le nom d'utilisateur et le mot de passe directement depuis le trafic, sans rien deviner et sans rien de journalisé sur le serveur. Les frappes sont envoyées caractère par caractère, donc l'identifiant est réassemblé à partir du flux, et le compte capturé est généralement réutilisable."
keywords:
  - sniffing de mot de passe
  - identifiants en clair
  - tcpdump
  - wireshark
  - telnet
---

# Sniffing de mot de passe

Telnet transmet le login en clair, de sorte que capturer une authentification réelle donne l'identifiant exact sans force brute ni bruit de login échoué sur le serveur. Le seul détail est que Telnet envoie l'entrée caractère par caractère (et le serveur en fait souvent l'écho), de sorte que le nom d'utilisateur et le mot de passe apparaissent comme un flux d'octets uniques à réassembler, ce que les outils standards font automatiquement. Un identifiant Telnet sniffé est le moyen le plus discret d'obtenir un accès, et le compte est généralement administratif sur l'appareil et réutilisé ailleurs.

```bash
# capture and reassemble a Telnet login from an on-path position
tcpdump -i eth0 -A port 23
# Wireshark: Follow TCP Stream reconstructs the full login (and session)
# scripted extraction of credentials from a pcap
tshark -r telnet.pcap -Y 'telnet' -T fields -e telnet.data | tr -d '\n'
```

## Notes d'exploitation

- Le réassemblage importe : parce que l'entrée est par caractère (et renvoyée en écho), lisez les octets client vers serveur pour récupérer ce qui a été tapé ; Follow TCP Stream de Wireshark et `tshark` le font directement.
- Cela capture l'identifiant réel sans rien de journalisé comme tentative échouée, bien plus discret que la [force brute](../authentication/password-brute-force.md), donc préférez-le dès qu'une position de capture existe.
- Le login capturé est généralement un compte administratif de l'appareil et couramment réutilisé sur d'autres systèmes et services ; testez-le largement.
- Nécessite une position on-path ou de tap (ARP/DNS/route sur un segment plat, un port SPAN, ou un intermédiaire compromis).

## Références

- [HackTricks: Telnet sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
