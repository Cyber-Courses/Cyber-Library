---
order: 1
title: "Énumération : cartographier l'inventaire et les services vSphere"
description: "Avec un accès à vCenter, un attaquant énumère l'inventaire complet via l'API vSphere : chaque hôte ESXi, machine virtuelle, datastore, réseau et rôle d'utilisateur. Non authentifié, les endpoints exposés et les chaînes de version identifient le build pour faire correspondre les vulnérabilités connues. Cette reconnaissance cadre le périmètre et sélectionne les cibles avant toute action destructrice."
keywords:
  - énumération vcenter
  - api vsphere
  - govc
  - inventaire
  - empreinte de version
---

# Énumération

La valeur de vCenter pour un attaquant est sa vue complète de l'environnement, et l'API vSphere l'expose. Authentifié, un attaquant liste chaque hôte, VM, datastore, réseau et permission au même endroit ; non authentifié, les endpoints de service exposés et les chaînes de version identifient le build pour le faire correspondre aux vulnérabilités connues. Cette reconnaissance cartographie le périmètre et choisit les cibles, les VM à forte valeur, les datastores contenant les disques sensibles, les hôtes à travers lesquels pivoter, avant d'agir.

## Empreinte non authentifiée

```bash
# version and build, to match advisories
curl -sk https://<vcenter>/sdk/vimServiceVersions.xml
curl -sk https://<vcenter>/analytics/telemetry/ph/api/hyper/send 2>/dev/null
# the VAMI appliance management interface on 5480 and its version
curl -sk https://<vcenter>:5480/
```

## Inventaire authentifié

```bash
# govc (vSphere CLI) against the API with credentials or a token
export GOVC_URL='https://<vcenter>' GOVC_USERNAME='administrator@vsphere.local' GOVC_PASSWORD='...' GOVC_INSECURE=1
govc about                                   # version/build
govc ls -l /                                 # datacenters, folders
govc find / -type h                          # every ESXi host
govc find / -type m                          # every VM
govc datastore.ls -l                         # datastores (disks to steal)
govc permissions.ls                          # roles and who holds them
```

## Notes d'exploitation

- La chaîne de version non authentifiée est la première chose à récupérer ; vCenter regroupe de nombreux services et son build mappe directement aux [exploits de gestion connus](known-management-exploits.md) applicables.
- Authentifié, l'inventaire sélectionne les cibles : trouvez les contrôleurs de domaine et les serveurs sensibles parmi les VM, et les datastores qui contiennent leurs disques pour un vol hors ligne.
- L'énumération des permissions révèle quels comptes détiennent des rôles administratifs, guidant le ciblage des identifiants et les voies d'abus du SSO.
- `govc` parle la même API que l'UI, donc un token ou un identifiant volé pilote une énumération complète de manière non interactive.

## Outils

- [govc (vSphere CLI)](https://github.com/vmware/govmomi/tree/main/govc)

## Références

- [vSphere Web Services API](https://developer.vmware.com/apis/vsphere-automation/latest/)
- [VMware vCenter documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
