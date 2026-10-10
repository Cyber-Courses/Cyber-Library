---
title: "Compute Engine"
description: "Attaquer les VM Compute Engine : métadonnées et injection de clé SSH, OS Login, et portées du compte de service attaché à l'instance."
keywords:
  - Compute Engine
  - GCE
  - metadata
  - SSH
  - OS Login
---

# Compute Engine

Une VM Compute Engine lie deux choses que recherche un attaquant : un moyen d'**exécuter du code** (via les métadonnées d'instance, auxquelles l'invité fait confiance pour les startup scripts et les clés SSH) et une **identité** (le compte de service attaché, dont le serveur de métadonnées distribue le jeton). Détenir `compute.instances.setMetadata` sur une instance équivaut à une exécution de code sur celle-ci ; atteindre le serveur de métadonnées depuis un shell sur celle-ci, c'est son compte de service.

## Ce qui se rattache ici

- **[Métadonnées et SSH](metadata-and-ssh.md)** : écrire un startup script ou une clé SSH pour obtenir un shell, et lire le serveur de métadonnées sur l'hôte.
- **[Portée du compte de service](service-account-scope.md)** : utiliser le compte de service attaché à l'instance et les portées OAuth pour appeler les API Google.

Créer une *nouvelle* VM avec un compte de service privilégié attaché relève de l'escalade [actAs compute-instance](../../identity/privilege-escalation/act-as-deployment/compute-instance.md) ; ces pages couvrent les instances existantes.

## Références

- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google: Compute Engine metadata](https://cloud.google.com/compute/docs/metadata/overview)
