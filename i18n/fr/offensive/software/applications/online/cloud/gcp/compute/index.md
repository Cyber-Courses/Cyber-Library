---
title: "GCP compute"
order: 3
description: "Attaquer le compute GCP : métadonnées, SSH et portées de compte de service de Compute Engine, et accès au cluster et identité de nœud GKE."
keywords:
  - Compute Engine
  - GKE
  - metadata
  - service account scope
  - SSH
---

# Compute

Le compute GCP est l'endroit où les **comptes de service** d'un projet sont le plus exposés : chaque VM Compute Engine et chaque nœud GKE s'exécute en tant que compte de service dont le serveur de métadonnées distribue le jeton sur demande, et la permission d'écrire les métadonnées d'une VM est la permission d'y exécuter du code. Les surfaces ci-dessous transforment le contrôle d'une instance, ou l'accès à celle-ci, en l'identité de cette instance.

## Ce qui se rattache ici

- **[Compute Engine](compute-engine/index.md)** : injection de métadonnées et de clé SSH, OS Login, et portées du compte de service attaché à l'instance.
- **[GKE](gke/index.md)** : récupérer les identifiants de cluster depuis le côté cloud, et voler le jeton du compte de service du node pool.

Déployer une *nouvelle* instance pour qu'elle s'exécute en tant que compte de service privilégié est une voie d'escalade de privilèges traitée sous [identité actAs](../identity/privilege-escalation/act-as-deployment/compute-instance.md) ; les pages ici couvrent les instances et clusters qui existent déjà. Les attaques Kubernetes génériques internes au cluster se trouvent dans la section Conteneurs et sont référencées en renvoi, non dupliquées.

## Références

- [HackTricks Cloud: GCP compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
