---
title: "Mouvement latéral : se propager dans un cluster et vers le cloud"
order: 5
description: "Après un point d'appui, le déplacement à travers un cluster Kubernetes suit ses relations de confiance : un jeton de compte de service utilisé contre l'API server, des secrets et jetons dérobés dans d'autres namespaces, le pivotement de pod à pod sur le réseau plat, et le suivi de la workload identity d'un pod vers le compte cloud environnant. Chaque étape réutilise des identifiants que le cluster distribue."
keywords:
  - lateral movement
  - service account token
  - pod pivoting
  - workload identity
  - kubernetes
---

# Mouvement latéral

Le mouvement latéral dans Kubernetes est une réutilisation d'identifiants le long des arêtes de confiance propres au cluster. Un pod détient un jeton de compte de service qui s'authentifie auprès de l'API server ; l'API expose des secrets et d'autres jetons ; le réseau plat des pods atteint directement les autres workloads ; et la workload identity cloud d'un pod atteint le compte cloud environnant. Chaque mouvement prend un identifiant que le cluster a émis dans un but légitime et l'utilise pour atteindre la cible suivante, de sorte que le déplacement nécessite rarement un exploit, seulement de l'énumération et de la réutilisation.

```bash
# the token you hold, what it reaches, and the network around you
kubectl auth can-i --list
kubectl get secrets --all-namespaces 2>/dev/null | head
kubectl get endpoints -A 2>/dev/null | head
```

## Sous-sujets

- **[Jeton de compte de service vers l'API](service-account-token-to-api.md)** : s'authentifier et agir avec un jeton de pod.
- **[Vol de jetons et de secrets](token-and-secret-theft.md)** : récolter des identifiants à travers les namespaces.
- **[Pivotement de pod à pod](pod-to-pod-pivoting.md)** : atteindre d'autres workloads sur le réseau plat.
- **[IAM cloud via workload identity](cloud-iam-via-workload-identity.md)** : suivre l'identité d'un pod jusque dans le compte cloud.

## Références

- [Kubernetes : contrôler l'accès](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [MITRE ATT&CK : matrice Containers](https://attack.mitre.org/matrices/enterprise/containers/)
- [HackTricks : mouvement latéral Kubernetes](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
