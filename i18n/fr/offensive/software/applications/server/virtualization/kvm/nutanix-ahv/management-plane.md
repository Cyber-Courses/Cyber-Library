---
order: 3
title: "Plan de gestion : l'API Prism et les services de la Controller VM"
description: "Nutanix est administré via Prism Element (par cluster) et Prism Central (multi-cluster), des API REST et des interfaces web servies par les Controller VM. L'accès à Prism contrôle chaque VM et la fabric de stockage : création et reconfiguration de VM, attachement et clonage de vdisks, et lecture des secrets du cluster, si bien que des identifiants Prism ou une faille d'API équivalent au contrôle du parc."
keywords:
  - prism
  - prism central
  - nutanix api
  - v3 api
  - management plane
---

# Plan de gestion

La gestion de Nutanix, c'est Prism : Prism Element s'exécute sur les Controller VM de chaque cluster pour la gestion locale, et Prism Central agrège de nombreux clusters. Les deux exposent des API REST (les interfaces v1/v2 et v3) et des interfaces web. Atteindre Prism avec des identifiants ou via une faille d'API équivaut au contrôle du parc de virtualisation : énumérer et manipuler chaque VM, créer ou reconfigurer des VM, attacher et cloner les vdisks qui contiennent les données des invités, gérer les images, et lire la configuration et les secrets du cluster. Prism Central est particulièrement précieux car il s'étend sur plusieurs clusters.

## Atteindre et piloter Prism

```bash
# Prism REST API (served by the CVM / Prism Central), typically on 9440
curl -sk -u '<user>:<pass>' https://<prism>:9440/api/nutanix/v2.0/cluster/
curl -sk -u '<user>:<pass>' https://<prism>:9440/api/nutanix/v3/vms/list -XPOST -d '{"kind":"vm"}'
# enumerate VMs, images, and storage; then act
curl -sk -u '<u>:<p>' https://<prism>:9440/api/nutanix/v3/images/list -XPOST -d '{"kind":"image"}'
```

## Actions de contrôle

```text
With Prism access an attacker can:
- create or reconfigure a VM to attach/clone another VM's vdisk and read it
- clone or snapshot vdisks and export them (data theft without entering a VM)
- change VM boot/network settings, or deploy an attacker VM on the cluster
- read cluster configuration, users, and integration credentials
Prism Central extends all of this across every registered cluster.
```

## Notes d'exploitation

- Les identifiants Prism équivalent au contrôle du parc ; l'API v3 en particulier expose la gestion des VM, des images et du stockage, si bien qu'attacher ou cloner le vdisk d'une VM cible lit ses données sans évasion depuis l'invité.
- Prism Central s'étend sur plusieurs clusters, donc le compromettre multiplie la portée ; traitez-le comme la cible de gestion la plus précieuse.
- Les API sont servies par les Controller VM, donc l'accès à la CVM (depuis [Accès à l'hôte et shell](host-access-and-shell.md)) et l'accès à Prism se renforcent mutuellement, et des identifiants de service locaux sur une CVM peuvent s'authentifier auprès de Prism.
- Nutanix publie des avis de sécurité pour Prism et les API ; identifiez la version et faites correspondre toute vulnérabilité du plan de gestion.

## Références

- [Référence de l'API Nutanix v3](https://www.nutanix.dev/api-reference/)
- [Documentation Prism](https://portal.nutanix.com/)
- [Avis de sécurité Nutanix](https://www.nutanix.com/support-services/security-advisories)
