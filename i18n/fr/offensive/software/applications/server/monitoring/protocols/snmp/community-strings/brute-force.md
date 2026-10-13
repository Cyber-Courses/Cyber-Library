---
title: "Brute force : attaques par dictionnaire contre la chaîne de communauté"
description: "SNMP n'a pas de verrouillage de compte et la chaîne de communauté est le seul identifiant, le brute force par dictionnaire est donc praticable. onesixtyone envoie des sondes UDP rapides sur de nombreuses chaînes et hôtes à la fois, et snmp-check ou hydra confirment les réussites. Balayer un dictionnaire de communautés sur un sous-réseau trouve rapidement chaque agent à chaîne faible."
keywords:
  - snmp brute force
  - onesixtyone
  - wordlist
  - no lockout
  - udp 161
---

# Brute force

Comme SNMP n'a aucune notion de compte ni de verrouillage et que la chaîne de communauté est l'unique identifiant, la brute forcer est simple et rapide. `onesixtyone` est conçu spécialement pour cela : il émet des requêtes SNMP UDP sur une liste de chaînes de communauté (et une liste d'hôtes) à haut débit et signale quelles chaînes suscitent une réponse, rendant praticable le balayage de tout un sous-réseau contre un dictionnaire de communautés en quelques secondes. Une chaîne récupérée est ensuite confirmée et utilisée avec les outils net-snmp. L'absence de verrouillage et le transport UDP sans connexion font que la seule vraie limite est la perte de paquets, les réessais importent donc.

```bash
# sweep community strings (and optionally many hosts) with onesixtyone
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <target>
onesixtyone -c comm.txt -i hosts.txt                     # many hosts at once
# confirm a hit and start reading
snmp-check -c <found> <target>
# hydra also brute-forces SNMP (slower, but integrates with other services)
hydra -P comm.txt -s 161 <target> snmp
```

## Notes d'exploitation

- `onesixtyone` est la voie rapide : il est piloté par le débit UDP et balaie à la fois les chaînes et les hôtes, il est donc idéal pour trouver chaque agent à chaîne faible sur un sous-réseau en une seule passe.
- Tenez compte de la perte UDP : utilisez des réessais et un débit raisonnable ; une réponse manquée n'est pas une chaîne en échec, retestez donc les échecs apparents avant de conclure.
- Il n'y a pas de verrouillage à déclencher, un grand dictionnaire est donc viable, mais un équipement peut quand même limiter le débit ou abandonner sous inondation ; ajustez le débit si les réponses se tarissent.
- Une chaîne trouvée pilote l'[énumération](../enumeration/index.md) ; brute forcez séparément pour une chaîne en lecture-écriture, car elle est souvent différente et permet l'[accès en écriture](../write-access.md).

## Outils

- [onesixtyone](https://github.com/trailofbits/onesixtyone)
- [snmp-check](https://gitlab.com/kalilinux/packages/snmpcheck)

## Références

- [HackTricks: SNMP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
