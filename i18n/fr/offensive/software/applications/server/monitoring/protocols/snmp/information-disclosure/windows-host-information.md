---
title: "Informations sur l'hôte Windows : données étendues via SNMP"
order: 3
description: "Là où le service SNMP est activé sur Windows, la MIB host-resources et l'ancienne MIB LanMgr exposent des données étendues : comptes d'utilisateurs locaux, processus en cours, logiciels installés, partages et services en écoute. Les parcourir profile l'hôte en profondeur, révélant les comptes à cibler, les logiciels à exploiter et le rôle du système, à partir d'une community string de lecture."
keywords:
  - windows snmp
  - host-resources mib
  - lanmgr mib
  - running processes
  - installed software
---

# Informations sur l'hôte Windows

Lorsque le service SNMP est installé et activé sur un hôte Windows, il expose bien plus que le groupe system à travers deux MIB. La MIB host-resources (`1.3.6.1.2.1.25`) est standard et riche : `hrSWRunName` liste les processus en cours, `hrSWInstalledName` liste les logiciels installés, `hrStorageTable` donne les disques et la mémoire, et `hrDeviceTable` le matériel. L'ancienne LanMgr-MIB-II (`1.3.6.1.4.1.77`) ajoute des données propres à Windows : `.1.2.25` énumère les comptes d'utilisateurs locaux, et d'autres sous-arbres exposent les partages et les rôles de domaine/serveur. Les parcourir profile l'hôte en profondeur, noms d'utilisateurs locaux à cibler pour des attaques de mots de passe, logiciels en cours et installés à faire correspondre à des exploits, partages auxquels accéder et rôle de la machine, le tout à partir d'une community string de lecture sans s'authentifier auprès de l'hôte.

```bash
# host-resources MIB: processes and installed software
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.25.4.2.1.2   # hrSWRunName (running processes)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.25.6.3.1.2   # hrSWInstalledName (installed software)
# LanMgr-MIB: local user accounts and shares
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1.77.1.2.25    # local user accounts
snmpbulkwalk -v2c -c public <target> 1.3.6.1.4.1.77.1.2.27    # shares
# snmp-check and snmpenum format the Windows data automatically
snmp-check -c public <target>
```

## Notes d'exploitation

- L'énumération des utilisateurs locaux (`1.3.6.1.4.1.77.1.2.25`) est directement utile : elle fournit une liste de noms d'utilisateurs validée pour du password spraying contre SMB, RDP ou WinRM sur le même hôte, obtenue sans toucher à ces services.
- Les processus en cours et les logiciels installés révèlent ce qu'il faut attaquer (un service vulnérable, un EDR à prendre en compte, une base de données) ainsi que le rôle de l'hôte ; la liste des processus peut aussi exposer des lignes de commande dans certaines configurations.
- Cela nécessite que la fonctionnalité SNMP de Windows soit activée, ce qui est moins courant sur les systèmes modernes mais fréquent sur les serveurs et appliances plus anciens ; un `sysDescr` confirmant Windows et une chaîne fonctionnelle sont le signal pour parcourir ces MIB.
- Injectez les noms d'utilisateurs et les logiciels dans les autres services de l'hôte ; combinez avec la [Divulgation du système et de la topologie](system-and-topology-disclosure.md) pour le contexte réseau.

## Références

- [RFC 2790 (Host Resources MIB)](https://datatracker.ietf.org/doc/html/rfc2790)
- [HackTricks: SNMP Windows](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
