---
title: "Syslog : attaquer l'ingestion de logs"
order: 2
description: "Syslog achemine les événements de log vers les collecteurs et les SIEM, généralement sur UDP 514 sans authentification, et le pipeline fait confiance à ce qu'il reçoit. Cette confiance est la surface d'attaque : usurper la source des enregistrements, injecter des entrées falsifiées et découpées par saut de ligne pour empoisonner l'enregistrement et exploiter les parsers en aval, et inonder le collecteur pour noyer les vrais événements ou perturber les alertes."
keywords:
  - syslog
  - udp 514
  - log injection
  - spoofing
  - siem
---

# Syslog

Syslog est le protocole standard d'acheminement des événements de log depuis les systèmes et les équipements vers un collecteur central ou un SIEM, classiquement sur UDP 514 sans authentification ni protection d'intégrité (des variantes TCP et TLS existent mais sont bien moins courantes). Tout le pipeline de supervision et de détection en aval fait confiance à ces enregistrements, et cette confiance est la surface d'attaque. Un attaquant sur le réseau usurpe l'adresse source des enregistrements pour attribuer des logs falsifiés à d'autres hôtes, injecte des entrées conçues et découpées par saut de ligne pour forger des événements et empoisonner les parsers et les règles de corrélation dont dépend le SIEM, et inonde le collecteur pour noyer le vrai signal ou perturber le pipeline. L'objectif est généralement de tromper les opérateurs, de masquer une activité ou d'exploiter un point de traitement de logs en aval.

```bash
# send an arbitrary syslog record (UDP 514)
logger -n <collector> -P 514 -d "test from attacker"
echo '<34>1 2025-01-01T00:00:00Z host app - - - forged event' | nc -u -w1 <collector> 514
```

## Sous-sujets

- **[Usurpation de source](source-spoofing.md)** : falsifier l'origine des enregistrements de log.
- **[Injection et falsification](injection-and-forgery.md)** : entrées falsifiées et abus des parsers en aval.
- **[Inondation et évasion](flooding-and-evasion.md)** : noyer et perturber le pipeline.

## Références

- [RFC 5424 (Syslog protocol)](https://datatracker.ietf.org/doc/html/rfc5424)
- [HackTricks: 514 syslog](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsyslog)
