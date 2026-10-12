---
title: "Inondation et évasion : noyer et perturber le pipeline de logs"
order: 3
description: "Les collecteurs syslog ont un débit et un stockage finis, donc un attaquant les inonde d'enregistrements en gros volume pour noyer les vrais événements dans le bruit, déclencher une rotation qui fait disparaître les preuves, épuiser le stockage ou submerger l'ingestion et les alertes du SIEM. L'inondation sert à masquer une activité au sein du pic et à dégrader la détection pendant qu'une attaque se déroule."
keywords:
  - log flooding
  - evasion
  - rotation
  - denial of service
  - siem
---

# Inondation et évasion

Un pipeline syslog, collecteur, stockage et ingestion SIEM, a une capacité finie, et un attaquant en abuse pour contourner et perturber. Envoyer un gros volume d'enregistrements noie les véritables événements d'une attaque dans un flot de bruit, de sorte que les vrais indicateurs sont enfouis parmi des milliers d'entrées bénignes ou fabriquées, faisant échouer l'examen manuel et automatisé. Une inondation soutenue force la rotation des logs et les limites de rétention à faire disparaître ou à écarter les enregistrements plus anciens, détruisant les preuves d'une activité antérieure. Elle peut épuiser le stockage du collecteur ou saturer l'ingestion au point que des logs légitimes soient perdus (de nombreux collecteurs UDP perdent silencieusement sous charge), créant des angles morts. Et elle peut submerger la corrélation et les alertes du SIEM, retardant ou supprimant la détection pendant que l'attaque se déroule.

```bash
# high-volume record generation to the collector (UDP, no handshake to slow it)
yes '<34>flood filler event' | head -1000000 \
  | while read l; do printf '%s\n' "$l"; done | nc -u <collector> 514
# or a tight loop / parallel senders to sustain rate and force rotation/drops
for i in $(seq 1 50); do (while :; do logger -n <collector> -P 514 noise; done &) ; done
```

## Notes d'exploitation

- Deux effets d'évasion : enfouir les vrais événements dans le bruit (pour qu'ils soient manqués à l'examen et à la corrélation) et forcer la rotation/rétention à écarter les preuves plus anciennes ; calez l'inondation pour couvrir l'activité que vous voulez masquer.
- Les collecteurs UDP perdent couramment de façon silencieuse sous charge, ce qui crée de véritables trous d'ingestion, des logs légitimes émis pendant l'inondation peuvent ne jamais être stockés, donc l'angle mort est réel, pas seulement bruyant.
- L'inondation est bruyante par nature et peut elle-même déclencher une alerte sur des anomalies de volume ; c'est un compromis, dégrader la détection et les preuves au prix d'un pic évident, elle convient donc à de courtes fenêtres pendant une opération plus bruyante.
- Combinez avec l'[injection et la falsification](injection-and-forgery.md) pour que l'inondation implante aussi des événements trompeurs, et avec l'[usurpation de source](source-spoofing.md) pour la mal attribuer.

## Références

- [RFC 5426 (UDP syslog, delivery not guaranteed)](https://datatracker.ietf.org/doc/html/rfc5426)
- [HackTricks: syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
