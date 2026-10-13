---
title: "Sniffing de mot de passe : capture des identifiants r-command"
description: "rexec envoie un nom d'utilisateur et un mot de passe en clair à chaque requête, et rlogin envoie un mot de passe en clair lorsque la confiance d'hôte ne s'applique pas. Un attaquant bien positionné les capture directement dans le trafic, obtenant des identifiants réutilisables sans aucune devinette et sans qu'aucune tentative échouée ne soit journalisée."
keywords:
  - sniffing de mot de passe
  - rexec
  - rlogin
  - identifiants en clair
  - capture
---

# Sniffing de mot de passe

Lorsque les r-commands utilisent un mot de passe, rexec toujours, rlogin quand la confiance d'hôte n'accorde pas l'accès, ce mot de passe est envoyé en clair : un attaquant bien positionné le capture donc directement dans le flux. rexec inclut le nom d'utilisateur et le mot de passe dans chaque requête d'exécution (chaque invocation est donc une occasion de capture), et l'invite de mot de passe de repli de rlogin est envoyée caractère par caractère puis reconstituée. Le résultat est un identifiant de compte réutilisable obtenu discrètement : pas de force brute, pas d'enregistrement d'échec de connexion, pas de verrouillage, et de tels identifiants sont généralement réutilisés dans tout l'environnement hérité.

```bash
# capture credentials from rexec (512) and rlogin (513)
tcpdump -i eth0 -A 'port 512 or port 513' -w creds.pcap
tshark -r creds.pcap -q -z follow,tcp,ascii,0     # username/password in the stream
```

## Notes d'exploitation

- rexec est la source la plus riche car il transmet l'identifiant avec chaque commande : un usage scripté/automatisé de rexec le divulgue donc de façon répétée ; rlogin ne le divulgue que lorsqu'il bascule sur un mot de passe (pas lorsque la confiance d'hôte s'applique).
- Reconstituez les octets client vers serveur (rlogin envoie caractère par caractère) pour récupérer le mot de passe saisi ; `tshark`/Wireshark le font automatiquement.
- Les identifiants capturés sont réutilisables sur l'hôte et sur les systèmes hérités ; testez largement, et préférez la capture à la devinette de type [force brute](../rlogin/cleartext-passwords.md) partout où une position existe.
- Les sessions rsh en confiance d'hôte ne transportent généralement aucun mot de passe (rien à sniffer de ce côté) ; ciblez rexec/rlogin pour les identifiants et rsh pour le [contenu/les commandes](command-interception.md).

## Références

- [HackTricks: r-commands sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
