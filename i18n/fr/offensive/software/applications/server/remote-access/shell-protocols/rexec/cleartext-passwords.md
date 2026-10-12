---
title: "Mots de passe en clair : capturer l'identifiant rexec"
order: 2
description: "rexec envoie le nom d'utilisateur et le mot de passe en clair dans le cadre de la requête d'exécution, de sorte qu'un attaquant bien positionné capture l'identifiant directement depuis le trafic. L'identifiant est réutilisable, et comme rexec le transmet à chaque commande, des captures répétées sont triviales partout où le service est utilisé."
keywords:
  - cleartext password
  - rexec
  - credential capture
  - sniffing
  - port 512
---

# Mots de passe en clair

`rexec` transporte le nom d'utilisateur et le mot de passe en clair dans la requête d'exécution envoyée à `rexecd`, de sorte qu'un attaquant bien positionné lit l'identifiant directement sur le fil. Parce que rexec envoie l'identifiant à chaque invocation de commande (ce n'est pas une session authentifiée persistante comme rlogin mais un échange par commande), chaque utilisation est une nouvelle occasion de capture. L'identifiant capturé est un véritable mot de passe de compte, réutilisable sur l'hôte et généralement à travers l'environnement hérité, obtenu sans aucune devinette et sans que rien ne soit journalisé comme un échec.

```bash
# capture the rexec credential from an on-path position
tcpdump -i eth0 -A port 512 -w rexec.pcap
tshark -r rexec.pcap -q -z follow,tcp,ascii,0     # the username/password appear in the request
```

## Notes d'exploitation

- L'identifiant est dans la requête elle-même, de sorte qu'une seule invocation rexec capturée le livre ; aucun défi ni hachage n'est impliqué.
- La transmission par commande signifie qu'un usage scripté/automatisé de rexec (tâches cron, scripts d'administration) laisse fuiter l'identifiant de manière répétée, rendant la capture facile là où rexec est en usage régulier.
- Le mot de passe capturé est un identifiant de compte système, réutilisable à travers les hôtes et parfois d'autres services ; testez-le largement.
- La capture est plus discrète que la force brute et évite tout verrouillage ; privilégiez-la là où une position sur le chemin existe, et combinez-la avec le [contournement d'hôte de confiance](trusted-host-bypass.md) là où la confiance supprime entièrement le besoin d'un mot de passe.

## Références

- [rexecd and the exec protocol](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
