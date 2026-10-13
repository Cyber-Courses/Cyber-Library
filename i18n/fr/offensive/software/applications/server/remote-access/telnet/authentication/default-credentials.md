---
title: "Identifiants par défaut : logins Telnet constructeur et d'usine"
description: "Telnet survit surtout sur des équipements, routeurs, switchs, caméras, imprimantes et IoT, livrés avec des identifiants par défaut fixes que les utilisateurs changent rarement. Essayer les valeurs par défaut documentées de l'équipement, ainsi que des paires génériques comme root:root et admin:admin, est l'attaque Telnet au rendement le plus élevé, et ces comptes sont généralement administratifs sur l'équipement."
keywords:
  - identifiants par défaut
  - telnet
  - iot
  - valeur par défaut constructeur
  - mot de passe d'usine
---

# Identifiants par défaut

Telnet est désormais surtout un protocole d'équipement, et les équipements sont livrés avec des identifiants par défaut fixes qui sont documentés, largement connus et rarement changés. Ainsi, la première attaque Telnet et celle au rendement le plus élevé est d'essayer les valeurs par défaut : paires génériques (`root:root`, `admin:admin`, `root:<vendor>`, mots de passe vides) et les identifiants documentés spécifiques à l'équipement identifié. Ces comptes sont généralement administratifs, un compte réseau capable de passer en mode enable ou un shell root sur du Linux embarqué, de sorte qu'une réussite sur un identifiant par défaut équivaut généralement à un contrôle total de l'équipement, et des compromissions massives d'IoT ont été bâties exactement sur ce principe.

```bash
# identify the device (banner), then try its documented defaults + generics
nc <target> 23
hydra -C iot-default-combos.txt telnet://<target>     # user:pass combo list
nxc telnet <target> -u defaults_u.txt -p defaults_p.txt --no-bruteforce   # paired
```

## Notes d'exploitation

- Identifiez d'abord l'équipement ([bannière](../enumeration/banner-grabbing.md)) ; les valeurs par défaut sont spécifiques au constructeur/modèle, donc l'identification transforme la devinette en recherche, et les paires génériques couvrent le reste.
- Utilisez des listes de combos appariées (user:pass documentés) plutôt que des produits croisés complets, pour la rapidité et la discrétion ; les listes de valeurs par défaut IoT (telles qu'utilisées par les vers qui se propagent via Telnet) sont très efficaces.
- Les comptes sont couramment administratifs, de sorte que le succès équivaut à un contrôle de l'équipement (changement de configuration, firmware, souvent un shell root), et non à un point d'appui à faibles privilèges.
- Les mots de passe vides ou nuls sont une valeur par défaut fréquente des équipements ; testez-les explicitement.

## Références

- [HackTricks: Telnet default credentials](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [SecLists default credentials](https://github.com/danielmiessler/SecLists)
