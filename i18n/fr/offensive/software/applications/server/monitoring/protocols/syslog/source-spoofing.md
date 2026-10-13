---
order: 1
title: "Usurpation de source : forger l'origine des enregistrements syslog"
description: "Le syslog UDP n'a aucune authentification et la source enregistrée d'un évènement est prise dans le paquet, si bien qu'un attaquant usurpe l'IP source pour attribuer des entrées de log falsifiées à un autre hôte. Cela incrimine d'autres systèmes, implante des preuves trompeuses, et, parce que le collecteur fait confiance à l'origine apparente, empoisonne la détection et la corrélation attribuées à l'hôte."
keywords:
  - usurpation de source
  - syslog udp
  - usurpation d'ip
  - origine falsifiée
  - attribution
---

# Usurpation de source

Le syslog UDP ne porte aucune authentification, et le collecteur attribue chaque enregistrement à la source dont il semble provenir, l'IP source du paquet et le champ hostname du message. Un attaquant sur un chemin réseau qui atteint le collecteur usurpe l'adresse source UDP, de sorte que les enregistrements falsifiés sont journalisés comme s'ils provenaient d'un autre hôte. Cela permet à un attaquant d'attribuer des évènements fabriqués à un système choisi, l'incriminant, implantant des preuves trompeuses, ou déclenchant la détection et la réponse propres à un hôte contre une machine innocente. Parce qu'une grande partie de la logique du SIEM repose sur l'hôte source, les enregistrements à l'origine usurpée corrompent la corrélation et peuvent orienter entièrement une investigation.

```bash
# spoof the UDP source address of a syslog packet (needs raw-socket capability)
# scapy example: forge a record that appears to come from 10.0.0.5
python3 - <<'PY'
from scapy.all import IP, UDP, Raw, send
pkt = IP(src="10.0.0.5", dst="<collector>")/UDP(sport=514, dport=514)/ \
      Raw(load="<34>1 2025-01-01T00:00:00Z victimhost sshd - - - Accepted password for admin")
send(pkt, verbose=0)
PY
# the message's own hostname field is also attacker-controlled and often trusted
```

## Notes d'exploitation

- Deux canaux d'attribution sont falsifiables : l'IP source UDP (usurpée au niveau du paquet) et le champ hostname à l'intérieur du message syslog ; les collecteurs peuvent faire confiance à l'un ou l'autre, donc réglez les deux sur l'hôte que vous voulez incriminer.
- L'attaque nécessite une portée réseau jusqu'au collecteur avec la capacité d'envoyer de l'UDP usurpé (raw sockets, et un chemin qui ne filtre pas les adresses sources usurpées) ; les réseaux internes plats le permettent couramment.
- Les enregistrements à l'origine usurpée empoisonnent la détection et la corrélation attribuées à l'hôte, utiles pour tromper les intervenants, générer de fausses alertes contre une cible, ou faire paraître une véritable activité comme venue d'ailleurs.
- À combiner avec l'[injection et la falsification](injection-and-forgery.md) pour contrôler aussi bien le contenu que l'origine des enregistrements implantés.

## Références

- [RFC 5426 (Syslog over UDP)](https://datatracker.ietf.org/doc/html/rfc5426)
- [HackTricks: syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
