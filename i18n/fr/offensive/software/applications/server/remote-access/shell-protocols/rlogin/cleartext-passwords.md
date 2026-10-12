---
title: "Mots de passe en clair : capturer la connexion rlogin"
order: 1
description: "Lorsque la confiance d'hôte ne s'applique pas, rlogin se replie sur une invite de mot de passe envoyée en clair. Un attaquant bien positionné capture le nom d'utilisateur et le mot de passe directement depuis le trafic, et toute la session interactive avec, livrant un identifiant réutilisable sans aucune devinette et sans que rien ne soit journalisé comme une tentative échouée."
keywords:
  - cleartext password
  - rlogin
  - sniffing
  - credential capture
  - session
---

# Mots de passe en clair

Si la vérification de confiance d'hôte n'accorde pas l'accès, `rlogin` demande un mot de passe, et comme tout dans les r-commands ce mot de passe traverse le réseau sans chiffrement. Un attaquant sur le chemin capture le nom d'utilisateur et le mot de passe directement depuis le flux, plus la session interactive complète qui suit. Cela donne un identifiant réutilisable sans aucun du bruit de la force brute, aucun enregistrement de connexion échouée, aucun verrouillage, et le compte capturé est généralement réutilisé à travers l'environnement hérité où les r-commands persistent.

```bash
# capture the cleartext rlogin login and session
tcpdump -i eth0 -A port 513 -w rlogin.pcap
tshark -r rlogin.pcap -q -z follow,tcp,ascii,0     # reassemble login + session
```

## Notes d'exploitation

- La capture bat la devinette : une connexion rlogin interceptée livre l'identifiant exact discrètement, donc privilégiez-la partout où une position sur le chemin existe, le même raisonnement que le sniffing de mot de passe Telnet.
- Toute la session est également capturée, exposant commandes, sorties et tout identifiant secondaire saisi, voir [capture de commande/session](../traffic-interception/index.md).
- L'identifiant est généralement réutilisable sur d'autres hôtes hérités et parfois d'autres services ; testez-le largement.
- Nécessite une position sur le chemin/en dérivation ; combinez avec l'abus de confiance (qui ne nécessite aucun mot de passe) selon que la confiance ou un mot de passe est en usage.

## Références

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
