---
order: 3
title: "NIC virtuelles : s'évader d'ESXi via les cartes e1000 et vmxnet3"
description: "ESXi propose les cartes réseau e1000 émulée et vmxnet3 paravirtuelle. Le pilote invité programme des anneaux de descripteurs d'émission et de réception en mémoire invitée que le processus vmx lit par DMA pour véhiculer les paquets. Des failles dans la gestion des descripteurs et des anneaux, y compris l'interface de commandes vmxnet3, donnent un accès hors limites dans le processus hôte depuis la pile réseau de l'invité."
keywords:
  - vmxnet3
  - e1000
  - anneau de descripteurs
  - dma
  - carte réseau
---

# NIC virtuelles

La carte réseau d'un invité est émulée : l'Intel e1000 héritée, ou la vmxnet3 paravirtuelle de VMware. Dans les deux cas, le pilote invité met en place des anneaux de descripteurs d'émission et de réception en mémoire invitée et programme le périphérique avec leurs adresses de base et longueurs via des registres MMIO ou d'E/S ; le processus `vmx` lit alors les descripteurs par DMA pour envoyer et recevoir les paquets. Parcourir ces anneaux et descripteurs contrôlés par l'invité dans le processus hôte est la surface d'évasion, et l'interface additionnelle de commandes et de configuration de vmxnet3 ajoute des structures que l'hôte analyse depuis la mémoire invitée.

## Piloter la carte

```bash
lspci -nn | grep -iE 'ethernet|network'    # e1000 or vmxnet3
dmesg | grep -iE 'e1000|vmxnet3'
# an attacker guest driver programs the ring base/length registers and writes
# descriptors directly, controlling lengths, flags, and buffer pointers
```

Pour l'e1000, le chemin d'émission lit les descripteurs TX dont la longueur et l'adresse de tampon sont contrôlées par l'invité ; les failles classiques viennent de la manière dont le modèle gère les longueurs spécifiées par les descripteurs et l'offload de segmentation, où une longueur utilisée pour une copie dépasse le tampon sous-jacent. Pour vmxnet3, l'invité configure le périphérique via une région de mémoire partagée et un registre de commande, définissant les files TX/RX, leurs tailles d'anneau et les paramètres d'offload ; l'hôte analyse cette configuration et les champs par descripteur, de sorte qu'une longueur d'anneau incohérente, un index de descripteur hors plage ou un champ d'offload forgé pilote l'hôte hors limites.

```c
// e1000: a TX descriptor with a length/offload combination the model copies
// without bounding against the mapped buffer -> OOB read into the host
// vmxnet3: a device configuration (via the command register + shared area) with
// a ring length or descriptor index the host trusts -> OOB access when it walks
```

## Notes d'exploitation

- L'hôte fait confiance à la base de l'anneau, aux longueurs et aux champs de descripteurs que l'invité programme ; les primitives fiables sont une longueur d'émission qui déborde le tampon source et un index ou une taille d'anneau de réception que l'hôte utilise sans vérification de bornes.
- L'interface de configuration de vmxnet3 est la cible la plus riche car l'hôte analyse une configuration de périphérique définie par l'invité (files, anneaux, offloads) en plus des descripteurs par paquet, élargissant la surface d'analyse.
- L'analyse du contenu des paquets (offload de checksum et de segmentation) ajoute un traitement côté hôte de champs d'en-tête choisis par l'invité, un autre endroit où les hypothèses de longueur cèdent.
- Une évasion fonctionnelle combine la primitive OOB avec une fuite et est spécifique au build ; le modèle e1000 est partagé avec d'autres hyperviseurs, donc les bugs e1000 ont souvent une pertinence inter-produits.

## Références

- [Intel e1000 software developer's manual](https://www.intel.com/content/www/us/en/support/products/1285/ethernet-products.html)
- [vmxnet3 driver (open-vm-tools)](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware network device research](https://www.zerodayinitiative.com/blog)
