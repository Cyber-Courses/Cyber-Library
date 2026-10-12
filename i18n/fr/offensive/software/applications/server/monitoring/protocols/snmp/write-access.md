---
title: "Accès en écriture : reconfigurer un équipement avec une community string de lecture-écriture"
order: 4
description: "Une community string SNMP de lecture-écriture permet à un attaquant d'émettre snmpset pour modifier les paramètres d'un équipement : désactiver des interfaces, altérer des routes et des entrées ARP, modifier les champs system et, sur les équipements compatibles, déclencher une copie de configuration vers ou depuis TFTP. Cela va de la perturbation à l'implantation d'une configuration piégée, transformant une chaîne d'écriture en contrôle de l'équipement."
keywords:
  - snmpset
  - read-write community
  - config change
  - ifadminstatus
  - config overwrite
---

# Accès en écriture

Une community string de lecture-écriture fait passer SNMP de la lecture au contrôle de l'équipement. `snmpset` écrit des OID, et selon les MIB inscriptibles de l'équipement cela va de la nuisance à la prise de contrôle. Les objets inscriptibles standard permettent à un attaquant de modifier les champs system (`sysContact`/`sysLocation`/`sysName`), de désactiver ou d'activer administrativement des interfaces (`ifAdminStatus`), et de modifier des entrées de routage et ARP, provoquant une perturbation ciblée ou une redirection de trafic. Sur les équipements réseau dotés de la config-copy MIB, l'accès en écriture permet en outre de copier une configuration vers un serveur TFTP (exfiltration) ou, en sens inverse, de copier une configuration fournie par l'attaquant depuis TFTP sur l'équipement, écrasant la configuration en cours ou de démarrage par une version piégée. Cette dernière capacité rend une chaîne de lecture-écriture équivalente à un contrôle administratif.

```bash
RW=private
# change a system field (proof of write, and config tampering)
snmpset -v2c -c $RW <target> 1.3.6.1.2.1.1.6.0 s "owned"        # sysLocation
# administratively shut an interface (disruption / traffic steering)
snmpset -v2c -c $RW <target> 1.3.6.1.2.1.2.2.1.7.<ifIndex> i 2  # ifAdminStatus = down(2)
# Cisco: copy an ATTACKER config FROM tftp ONTO the device (backdoor the config)
#   same CISCO-CONFIG-COPY-MIB as exfiltration, with SourceFileType=networkFile(1)
#   and DestFileType=runningConfig(4) -> loads attacker's cfg into running-config
```

## Notes d'exploitation

- Confirmez la chaîne d'écriture avec un `snmpset` inoffensif (réaffecter à `sysLocation` sa valeur actuelle) avant toute action perturbatrice ; les chaînes de lecture et d'écriture diffèrent, donc une chaîne de lecture n'implique pas l'écriture.
- L'usage à plus fort impact sur le matériel Cisco et similaire est la config-copy MIB : exfiltrer la configuration pour en tirer des identifiants ([Exfiltration de configuration Cisco](information-disclosure/cisco-configuration-exfiltration.md)), ou charger une configuration d'attaquant pour ajouter un compte, ouvrir un accès d'administration ou rediriger le trafic, ce qui constitue une compromission complète de l'équipement.
- Les écritures d'interface et de routage permettent un déni de service ciblé et un man-in-the-middle (désactiver un lien, injecter une route), utiles mais plus bruyantes et plus perturbatrices que le vol d'identifiants.
- Les objets inscriptibles varient selon l'équipement et la MIB ; parcourez à la recherche d'OID inscriptibles et consultez les MIB de l'équipement, et rappelez-vous que les modifications de configuration peuvent être journalisées et nécessiter un équivalent de `write memory` pour être persistées.

## Références

- [net-snmp snmpset](http://www.net-snmp.org/docs/man/snmpset.html)
- [HackTricks: SNMP write](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
