---
order: 3
title: "Identifiants par défaut des BMC : identifiants admin livrés sur les contrôleurs de gestion"
description: "Les contrôleurs de gestion de carte mère (BMC) sont livrés avec des identifiants par défaut bien connus, ADMIN/ADMIN sur de nombreuses cartes Supermicro, root/calvin sur Dell iDRAC, et équivalents chez les constructeurs, rarement changés parce que le BMC est sur un réseau de gestion séparé. Ces identifiants par défaut accordent le contrôle administratif complet de l'alimentation, de la console et des médias virtuels de l'hôte."
keywords:
  - identifiants par défaut bmc
  - idrac root calvin
  - admin admin
  - supermicro
  - contrôleur de gestion
---

# Identifiants par défaut des BMC

Les BMC sont livrés avec des identifiants par défaut fixes, et ils sont changés bien moins souvent que les mots de passe d'OS ou d'application parce que le contrôleur de gestion vit sur un réseau séparé que les administrateurs considèrent comme privé et oublient. Les identifiants par défaut sont bien connus par constructeur : `root`/`calvin` sur Dell iDRAC, `ADMIN`/`ADMIN` sur de nombreuses cartes Supermicro, `Administrator` avec un mot de passe par appareil sur HPE iLO (parfois imprimé sur une languette amovible et laissé inchangé), et d'autres. Les essayer accorde le contrôle administratif complet du contrôleur, et donc de l'alimentation, de la console physique et des médias virtuels de l'hôte, de sorte qu'un BMC joignable avec des identifiants par défaut est une compromission complète du serveur.

```bash
# identify the BMC vendor, then try its documented defaults over IPMI and the web UI
ipmitool -I lanplus -H <bmc> -U root -P calvin user list          # Dell iDRAC default
ipmitool -I lanplus -H <bmc> -U ADMIN -P ADMIN user list          # common Supermicro default
curl -sk https://<bmc>/ | grep -iE 'idrac|ilo|supermicro'         # fingerprint for the web UI
```

## Notes d'exploitation

- Empreintez d'abord le constructeur (interface web, device ID IPMI), puis essayez les identifiants par défaut documentés de ce constructeur ; `root`/`calvin` (Dell), `ADMIN`/`ADMIN` (Supermicro) sont les plus rentables.
- Le fait que le BMC soit sur un réseau de gestion « privé » est la raison pour laquelle les identifiants par défaut persistent ; une fois ce réseau atteint (un segment plat, un hôte de rebond, un saut de VLAN), les identifiants par défaut sont la voie la plus facile.
- L'accès admin par défaut donne le contrôle de l'alimentation, la console KVM et les médias virtuels, suffisant pour démarrer une image d'attaquant et s'approprier l'OS, voir [iDRAC et iLO](idrac-and-ilo.md).
- À combiner avec la [divulgation de hash RAKP](ipmi-rakp-hash-disclosure.md) (qui récupère souvent exactement ces mots de passe par défaut) et le [cipher 0](ipmi-cipher-0-bypass.md) là où les identifiants par défaut ont été changés.

## Références

- [Dell iDRAC default credentials](https://www.dell.com/support/)
- [HackTricks: IPMI/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
