---
title: "Kubernetes : attaquer le cluster depuis un pod jusqu'au contrôle total"
description: "Attaquer Kubernetes de bout en bout : énumérer le cluster depuis un pod compromis, atteindre les composants exposés, escalader via le RBAC et les jetons de comptes de service, s'échapper d'un pod vers son nœud, se déplacer entre les workloads et vers le cloud, et persister dans le cluster."
keywords:
  - kubernetes
  - k8s attack
  - RBAC
  - pod escape
  - service account token
---

# Kubernetes

La plupart des attaques Kubernetes partent d'un unique pod compromis et montent en puissance. Un pod porte une identité (un jeton de compte de service), se trouve sur un réseau plat et s'exécute sur un nœud qui détient des identifiants pour l'ensemble du cluster. Le chemin est classique : énumérer, atteindre les composants exposés, escalader via le RBAC, s'échapper vers le nœud, se déplacer latéralement et persister. S'échapper du pod vers son nœud utilise les primitives indépendantes du runtime décrites dans [Évasion de conteneur](../../container-escape/index.md) ; ce domaine est le modèle d'attaque au niveau du cluster qui les entoure.

## Premiers gestes depuis un pod

Ces quelques commandes déterminent quelle branche du domaine s'applique ensuite :

```bash
# 1. identity and what it can do (drives the RBAC and lateral branches)
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo "$T" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool   # whoami
kubectl --token="$T" --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  --server=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT auth can-i --list

# 2. am I in an over-privileged pod? (drives pod-escape-to-node)
grep -E 'CapEff|Seccomp' /proc/self/status; cat /proc/self/uid_map
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup' | head   # hostPath

# 3. what is reachable? (drives exposed-components, network, cloud)
curl -sk https://$KUBERNETES_SERVICE_HOST/version
curl -s http://169.254.169.254/latest/meta-data/ 2>/dev/null | head   # cloud metadata
```

Lisez les résultats : une sortie `can-i` étendue pointe vers l'[Escalade de privilèges RBAC](rbac-privilege-escalation/index.md) ; un pod privilégié ou montant des chemins de l'hôte pointe vers l'[Évasion de pod vers le nœud](pod-escape-to-node/index.md) ; un kubelet, un etcd ou un dashboard accessible pointe vers les [Composants exposés](exposed-components/index.md) ; un endpoint de métadonnées accessible pointe vers le cloud, via [Métadonnées cloud depuis un pod](cluster-enumeration/cloud-metadata-from-pod.md).

## Sous-sujets

- **[Énumération du cluster](cluster-enumeration/index.md)** : cartographier le cluster et votre identité depuis un pod.
- **[Composants exposés](exposed-components/index.md)** : composants du plan de contrôle et des nœuds non authentifiés ou accessibles.
- **[Escalade de privilèges RBAC](rbac-privilege-escalation/index.md)** : transformer des permissions limitées en davantage.
- **[Évasion de pod vers le nœud](pod-escape-to-node/index.md)** : sortir d'un pod vers son nœud.
- **[Mouvement latéral](lateral-movement/index.md)** : se déplacer entre les workloads, les namespaces et vers le cloud.
- **[Persistance](persistence/index.md)** : conserver l'accès à travers les redémarrages et la remédiation.
- **[Réseau](network/index.md)** : attaquer le réseau du cluster et le service mesh.
- **[Chaîne d'approvisionnement](supply-chain/index.md)** : Helm, operators et admission comme voies de déploiement.

## Références

- [Concepts de sécurité Kubernetes](https://kubernetes.io/docs/concepts/security/)
- [MITRE ATT&CK : matrice Containers](https://attack.mitre.org/matrices/enterprise/containers/)
