---
order: 1
title: "vCenter : attaquer le plan de gestion vSphere"
description: "vCenter Server gère une flotte d'hôtes ESXi et constitue le plan de contrôle central d'un environnement vSphere. Le compromettre donne le contrôle de chaque hôte et VM gérés. La surface comprend l'énumération de l'inventaire et des services, le système d'identité et de tokens Single Sign-On, et un historique de vulnérabilités pré-authentification des services de gestion."
keywords:
  - vcenter
  - vsphere
  - sso
  - plan de gestion
  - flotte esxi
---

# vCenter

vCenter Server est le plan de gestion d'un environnement vSphere : il inventorie et contrôle de nombreux hôtes ESXi et toutes leurs machines virtuelles via une UI web et une API. Compromettre vCenter revient à compromettre tout le périmètre, car il peut exécuter des commandes dans les invités, accéder à chaque datastore et administrer chaque hôte. La surface offensive est constituée des services de gestion eux-mêmes : énumérer l'inventaire et les endpoints exposés, abuser du système d'identité et de tokens Single Sign-On, et exploiter les vulnérabilités pré-authentification récurrentes dans les nombreux services groupés de vCenter.

```bash
# vCenter exposes a web UI/API and several supporting services
nmap -p 443,5480,902,2012,2014 <vcenter>        # UI/API, VAMI, authd, vpxd services
curl -sk https://<vcenter>/ui/                   # vSphere client
curl -sk https://<vcenter>/sdk/vimServiceVersions.xml   # API version/discovery
```

## Sous-sujets

- **[Énumération](enumeration.md)** : cartographier l'inventaire, les hôtes et les services.
- **[Abus du SSO et des tokens](sso-and-token-abuse.md)** : le système d'identité et de tokens Single Sign-On.
- **[Exploits de gestion connus](known-management-exploits.md)** : les vulnérabilités pré-auth récurrentes des services.

## Références

- [VMware vCenter Server documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [vSphere Web Services API reference](https://developer.vmware.com/apis/vsphere-automation/latest/)
