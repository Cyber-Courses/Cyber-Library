---
title: "Énumération intra-pod : inventorier ce qu'un pod détient déjà"
order: 1
description: "Un pod compromis porte son propre jeton de compte de service, tous les secrets et configmaps montés sous forme de fichiers ou injectés comme variables d'environnement, et les détails de connexion des services avec lesquels il dialogue. Les lire en premier fournit fréquemment des identifiants de base de données, des clés d'API et un jeton qui s'authentifie auprès de l'API Kubernetes, avant tout appel externe."
keywords:
  - énumération de pod
  - jeton de compte de service
  - secrets kubernetes
  - variables d'environnement
  - configmap
---

# Énumération intra-pod

Avant de contacter le serveur d'API, inventoriez le pod lui-même. Kubernetes injecte l'identité et la configuration d'un pod dans le conteneur : le jeton de compte de service à un chemin fixe, les secrets et configmaps montés sous forme de fichiers ou de variables d'environnement, et les adresses des services que le pod utilise. Ces artefacts locaux constituent souvent le gain le plus rapide, car les secrets applicatifs et un jeton d'API exploitable se trouvent dans le système de fichiers et l'environnement.

```bash
# service-account identity: token, cluster CA, and namespace
cat /var/run/secrets/kubernetes.io/serviceaccount/token; echo
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace; echo
# environment variables: injected secrets and service addresses
env | grep -iE 'TOKEN|SECRET|PASSWORD|KEY|_HOST|_PORT|DATABASE|REDIS|AWS|AZURE|GCP'
# secrets/configmaps mounted as files (volume mounts)
mount | grep -E 'secret|configmap'
find / -maxdepth 6 -type d -path '*secret*' 2>/dev/null
# well-known credential files that get mounted in
find / -name '*.kubeconfig' -o -name 'id_rsa' -o -name '.dockerconfigjson' 2>/dev/null
```

## Ce que chaque source fournit

```bash
# the SA token authenticates to the API as the pod's identity (next pages)
# env-injected secrets are the app's own credentials to backends
# mounted secret volumes expose the raw secret values as files
cat /var/run/secrets/*/*/username /var/run/secrets/*/*/password 2>/dev/null
```

## Notes d'exploitation

- Lisez d'abord le jeton de compte de service et l'environnement injecté : le jeton est la clé de l'API, et les variables d'environnement contiennent couramment des identifiants de backend injectés au moment du déploiement.
- Les secrets montés en tant que volumes apparaissent sous forme de fichiers en clair sous leur chemin de montage ; `mount | grep secret` les localise sans deviner les chemins.
- Un kubeconfig ou un dockerconfigjson trouvé monté dans un pod est une escalade immédiate (identifiants de cluster ou de registre) ; triez-les avant toute autre chose.
- Reportez les découvertes dans [Découverte de comptes de service et de jetons](service-account-and-token-discovery.md) et [Énumération du serveur d'API](api-server-enumeration.md).

## Références

- [Kubernetes : jetons de compte de service](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes : distribuer des secrets aux pods](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks : Kubernetes enumeration](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
