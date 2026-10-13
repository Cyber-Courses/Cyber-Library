---
order: 2
title: "ESXi : attaquer l'hyperviseur bare-metal de VMware"
description: "ESXi exécute les machines virtuelles directement sur le matériel sous le vmkernel, chaque VM étant soutenue par un processus vmx. La surface offensive est l'évasion guest-to-host via les périphériques émulés, le vol des disques virtuels et des données depuis les datastores, l'obtention d'un shell hôte via les services de gestion, et les exploits d'évasion nommés récurrents issus de failles de modèles de périphériques."
keywords:
  - esxi
  - vmkernel
  - vmx
  - datastore
  - hyperviseur
---

# ESXi

ESXi est l'hyperviseur de type 1 de VMware : le `vmkernel` s'exécute sur le matériel, et chaque machine virtuelle est soutenue par un processus `vmx` en espace utilisateur qui émule ses périphériques. Offensivement, il est attaqué dans deux directions. Depuis l'intérieur d'un invité, la cible est l'émulation de périphériques de `vmx`, une évasion de sûreté mémoire vers l'hôte. Depuis l'hôte ou le réseau de gestion, les cibles sont les datastores qui contiennent les disques de chaque VM et les services de gestion qui donnent un shell hôte. ESXi est une cible à forte valeur car un seul hôte exécute de nombreuses VM, donc la compromission de l'hôte les expose toutes.

```bash
# from a guest: emulated device inventory (escape surface)
lspci -nn; lsusb
# from the network: ESXi management surface
nmap -p 22,443,427,902 <esxi-host>        # SSH, host UI/API, CIM/SLP, vmx authd
curl -sk https://<esxi-host>/            # host client / API reachable
```

## Sous-sujets

- **[Évasion guest-to-host](guest-to-host-escape/index.md)** : sortir d'une VM via les périphériques émulés.
- **[Vol de datastore et de VMDK](datastore-and-vmdk-theft.md)** : voler les disques virtuels et leurs données.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : obtenir l'exécution sur l'hôte ESXi.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les évasions récurrentes de modèles de périphériques.

## Références

- [VMware ESXi documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
