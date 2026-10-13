---
title: "Falsification d'enregistrements de flux : forger les enregistrements de flux exportés"
description: "L'export de flux est de l'UDP non authentifié, de sorte qu'un attaquant pouvant émettre vers le collecteur forge des enregistrements NetFlow/IPFIX. Les flux fabriqués injectent de fausses conversations dans le tableau de supervision, peuvent servir à masquer du trafic réel dans le bruit ou à attribuer à tort une activité, et peuvent cibler des bugs dans l'analyse des modèles et des enregistrements du collecteur, qui a produit des vulnérabilités de corruption de mémoire."
keywords:
  - flow spoofing
  - netflow forgery
  - ipfix template
  - collector parsing
  - false traffic
---

# Falsification d'enregistrements de flux

NetFlow, sFlow et IPFIX exportent les enregistrements en UDP sans authentification, de sorte qu'un collecteur accepte des enregistrements de flux de quiconque peut l'atteindre. Un attaquant forge donc des enregistrements : en concevant des datagrammes de flux qui décrivent des conversations qui n'ont jamais eu lieu. Les flux fabriqués empoisonnent le tableau de supervision, injectant du faux trafic pour tromper la planification de capacité et l'analytique de sécurité, noyant ou attribuant à tort les flux réels, et plantant des preuves de connexions qui incriminent un hôte ou masquent une activité dans le bruit. IPFIX et NetFlow v9 sont basés sur des modèles (l'exportateur envoie d'abord un modèle décrivant la disposition de l'enregistrement, puis des enregistrements de données y faisant référence), et analyser ces modèles et enregistrements contrôlés par l'attaquant est une surface en soi : des modèles malformés et des champs de longueur ont produit des vulnérabilités de corruption de mémoire et de déni de service dans les collecteurs.

```bash
# forge a NetFlow/IPFIX datagram to the collector (UDP); craft header + records
# scapy has NetflowHeader/NetflowRecord layers; example sketch:
python3 - <<'PY'
from scapy.all import IP, UDP, send
from scapy.layers.netflow import NetflowHeader, NetflowHeaderV5, NetflowRecordV5
pkt = IP(dst="<collector>")/UDP(sport=1234, dport=2055)/ \
      NetflowHeader()/NetflowHeaderV5(count=1)/ \
      NetflowRecordV5(src="10.0.0.5", dst="10.0.0.9", dpkts=1000, doctets=1000000)
send(pkt, verbose=0)
PY
# crafted IPFIX/v9 templates with inconsistent field lengths probe collector parsing bugs
```

## Notes d'exploitation

- Les flux falsifiés trompent tout ce qui est bâti sur les données : tableaux de bord de capacité, références de base et détection basée sur les flux, la falsification sert donc à fabriquer du trafic (incriminer un hôte, masquer des flux réels dans le bruit) ou à créer de faux indicateurs.
- Le collecteur fait confiance à la source et au modèle de l'exportateur ; le port 2055/4739 (NetFlow/IPFIX courant) en UDP n'a pas d'authentification, l'accessibilité est donc la seule condition préalable.
- L'analyseur de modèle/enregistrement est une surface de sûreté mémoire : des modèles IPFIX/NetFlow v9 conçus avec de mauvaises longueurs ou décomptes de champs ont fait planter ou ont exploité des collecteurs, un écouteur de flux est donc aussi une cible d'exploitation, associez le logiciel du collecteur à ses avis de sécurité.
- Combinez avec la [collecte des données du collecteur](collector-data-harvesting.md) : lisez d'abord la carte réelle, puis forgez des flux qui se fondent dans la masse ou induisent en erreur.

## Outils

- [Scapy (NetFlow layers)](https://scapy.net/)

## Références

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [NetFlow v9 (RFC 3954)](https://datatracker.ietf.org/doc/html/rfc3954)
