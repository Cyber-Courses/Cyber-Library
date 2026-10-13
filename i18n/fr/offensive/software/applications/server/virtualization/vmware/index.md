---
order: 2
title: "VMware : attaquer ESXi, vCenter et les hyperviseurs de bureau"
description: "La pile de virtualisation de VMware couvre l'hyperviseur bare-metal ESXi, le plan de gestion vCenter qui contrôle des flottes d'hôtes, et les produits de bureau Workstation et Fusion. Les thèmes offensifs sont constants : évasions guest-to-host via du code d'émulation de périphériques partagé, vol de disques virtuels et compromission du plan de gestion."
keywords:
  - vmware
  - esxi
  - vcenter
  - workstation
  - fusion
---

# VMware

Les produits de VMware partagent une lignée et une grande partie de leur code, donc leurs surfaces d'attaque se font écho. ESXi est l'hyperviseur de type 1 exécutant des VM sur du matériel nu ; vCenter est le plan de gestion contrôlant de nombreux hôtes ESXi ; et Workstation et Fusion sont les hyperviseurs de bureau de type 2 sous Windows et macOS. Le même code d'émulation de périphériques (SVGA, USB, NIC, les canaux GuestRPC et VMCI) apparaît dans tous, de sorte qu'une classe de bugs d'évasion guest-to-host dans l'un s'applique souvent aux autres. À côté des évasions, les cibles récurrentes sont le vol de disques virtuels et le plan de gestion.

```bash
# which VMware product and build is in front of you?
vmware -v 2>/dev/null                              # Workstation/Fusion/ESXi version
esxcli system version get 2>/dev/null              # ESXi
curl -sk https://<host>/sdk/vimServiceVersions.xml # vCenter/ESXi API
```

## Sous-sujets

- **[ESXi](esxi/index.md)** : l'hyperviseur bare-metal et ses évasions guest-to-host.
- **[vCenter](vcenter/index.md)** : le plan de gestion vSphere.
- **[Workstation et Fusion](workstation-and-fusion/index.md)** : les hyperviseurs de bureau.

## Références

- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware vSphere documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
