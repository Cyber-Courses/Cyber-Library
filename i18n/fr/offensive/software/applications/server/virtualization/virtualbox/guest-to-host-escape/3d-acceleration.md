---
order: 2
title: "Accélération 3D : s'évader de VirtualBox via le chemin graphique accéléré"
description: "L'accélération 3D de VirtualBox expose un canal de commandes graphiques depuis l'invité (via les Guest Additions) vers le processus de VM hôte, historiquement le passthrough OpenGL basé sur Chromium et le périphérique VMSVGA. L'hôte analyse un flux de commandes et de shaders contrôlé par l'invité, si bien que des failles dans cette analyse donnent des écritures hors limites dans le processus de VM, l'une des surfaces d'évasion les plus productives de VirtualBox."
keywords:
  - 3d acceleration
  - vmsvga
  - chromium
  - opengl
  - shader
---

# Accélération 3D

Lorsque l'accélération 3D est activée, l'invité VirtualBox (via les Guest Additions) envoie un flux de commandes graphiques au processus de VM hôte, qui l'interprète pour rendre avec le GPU hôte. Deux implémentations ont existé : l'ancien passthrough de commandes OpenGL basé sur Chromium, et le périphérique VMSVGA. Les deux analysent un flux riche contrôlé par l'invité, des paquets de commandes OpenGL ou des commandes SVGA/3D avec surfaces, shaders et buffers, dans le processus hôte, faisant de la 3D l'une des surfaces d'évasion les plus productives de VirtualBox, avec des bugs répétés de hors-limites et de confusion de type.

## La surface

```c
// with 3D enabled, the guest submits graphics commands over the Additions channel.
// Chromium path: HGCM-delivered OpenGL command packets with guest-chosen opcodes,
//   array sizes, and buffer lengths the host parser trusts.
// VMSVGA path: an SVGA FIFO/command stream (surfaces, shaders, DMA via GMRs) similar
//   to the VMware SVGA model. Primitives:
//    - a command with a length/count the host uses without bounding -> OOB write
//    - a shader or surface defined with sizes that overflow a host allocation
//    - an object id referenced across commands without validation -> type confusion
```

```bash
# 3D must be enabled on the VM (VBoxManage modifyvm <vm> --accelerate3d on) and the
# Guest Additions 3D driver loaded; confirm in the guest:
lsmod | grep -i vboxvideo; glxinfo 2>/dev/null | head
```

## Notes d'exploitation

- La 3D doit être activée et le pilote graphique des Guest Additions chargé pour la surface la plus riche ; c'est courant sur les VM de bureau configurées pour les graphiques.
- Le chemin VMSVGA reflète le modèle 3D SVGA de VMware (commandes FIFO, surfaces, shaders, DMA GMR), si bien que les classes de bugs et l'analyse [VMware SVGA](../../vmware/esxi/guest-to-host-escape/svga-and-3d-graphics.md) se transposent ; le chemin Chromium est un analyseur de commandes OpenGL avec ses propres bugs de tableau/longueur.
- Les primitives atterrissent dans le processus de VM ; une évasion complète associe une fuite à l'écriture et est spécifique à une version.
- Le flux de commandes est piloté depuis un pilote graphique invité contrôlé, offrant un contrôle précis des structures analysées.

## Références

- [Manuel VirtualBox : accélération 3D](https://www.virtualbox.org/manual/)
- [Zero Day Initiative : recherche VirtualBox 3D](https://www.zerodayinitiative.com/blog)
- [Alertes de sécurité Oracle](https://www.oracle.com/security-alerts/)
