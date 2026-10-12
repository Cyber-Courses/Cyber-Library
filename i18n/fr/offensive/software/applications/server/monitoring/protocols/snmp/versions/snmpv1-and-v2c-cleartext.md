---
title: "SNMPv1 et v2c en clair : capturer la community string"
order: 2
description: "SNMPv1 et v2c envoient la community string en clair dans chaque requête et n'offrent aucun chiffrement, de sorte qu'un attaquant bien positionné capture la chaîne en interceptant le trafic UDP 161 et lit passivement toutes les données SNMP. La chaîne capturée est ensuite réutilisée directement, et l'exposition s'applique même lorsqu'une v3 plus robuste est aussi configurée mais que v1/v2c reste activée."
keywords:
  - snmpv1
  - snmpv2c
  - cleartext
  - sniffing
  - community capture
---

# SNMPv1 et v2c en clair

SNMPv1 et v2c n'offrent aucune confidentialité : la community string est incluse en clair dans chaque paquet de requête, et les données des réponses ne sont pas chiffrées. Un attaquant bien positionné met donc en échec ces versions de façon passive, en capturant le trafic SNMP (une station de supervision interrogeant un équipement, ou un équipement envoyant des traps) et en lisant la community string directement dans les paquets, après quoi il la réutilise. Même les données sont exposées à un intercepteur sans qu'il ait besoin de la chaîne. Cela compte surtout là où les administrateurs ont activé SNMPv3 mais laissé v1/v2c activée pour de la supervision héritée : le chemin en clair sape le plus robuste, car les mêmes données d'équipement (et souvent le même accès effectif) sont atteignables via la version faible.

```bash
# capture SNMP traffic and read the community string from the packets
tcpdump -i eth0 -A udp port 161 or udp port 162          # requests (161) and traps (162)
# Wireshark: the "snmp.community" field shows the string in each v1/v2c PDU
tshark -i eth0 -Y 'snmp.community' -T fields -e snmp.community -e ip.dst | sort -u
# then reuse the captured string
snmpbulkwalk -v2c -c <captured> <target>
```

## Notes d'exploitation

- La community string figure en clair dans chaque paquet v1/v2c, donc une seule requête capturée (depuis une station de supervision qui interroge ou n'importe quel client) la fournit ; aucun cassage n'est en jeu.
- La capture nécessite une position sur le chemin ou en tap là où le SNMP circule (près d'un serveur de supervision, un port SPAN ou un segment partagé) ; le trafic de supervision vers de nombreux équipements y converge souvent, exposant de nombreuses chaînes à la fois.
- La présence de v1/v2c aux côtés de v3 est la faille exploitable : attaquez la version faible et ignorez entièrement v3.
- Une chaîne capturée alimente l'[énumération](../enumeration/index.md) et, si elle est en lecture-écriture, l'[accès en écriture](../write-access.md) ; elle est aussi fréquemment réutilisée dans tout le parc.

## Références

- [RFC 1157 (SNMPv1)](https://datatracker.ietf.org/doc/html/rfc1157)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
