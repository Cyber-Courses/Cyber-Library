---
title: "Espace de noms utilisateur de l'hôte : quand le root du conteneur est le root de l'hôte"
order: 4
description: "Si un conteneur n'est pas dans un espace de noms utilisateur séparé, l'UID 0 à l'intérieur est l'UID 0 sur l'hôte, de sorte que toute capacité ou tout accès aux fichiers obtenu dans le conteneur s'applique avec un privilège hôte réel. Partager un espace de noms utilisateur, ou ne pas en utiliser, est ce qui fait que les autres voies d'évasion donnent un véritable root hôte plutôt qu'une identité confinée et remappée."
keywords:
  - espace de noms utilisateur
  - uid_map
  - root de l'hôte
  - conteneur rootful
  - évasion de conteneur
---

# Espace de noms utilisateur de l'hôte

L'espace de noms utilisateur décide si le root du conteneur est un véritable root. Lorsqu'un conteneur s'exécute sans son propre espace de noms utilisateur (le comportement par défaut de Docker rootful et de la plupart des pods Kubernetes), l'UID 0 à l'intérieur du conteneur est l'UID 0 sur l'hôte, et les capacités que détient un processus sont des capacités de l'hôte. Lorsqu'un espace de noms utilisateur est en usage, l'UID 0 du conteneur est mappé vers un UID hôte non privilégié, et les capacités ne sont valides qu'à l'intérieur de cet espace de noms, de sorte que la même primitive d'évasion donne une identité remappée et confinée plutôt qu'un root hôte. Cette page porte sur la reconnaissance de la situation dans laquelle vous vous trouvez, car elle détermine si chaque autre voie de ces pages donne réellement un root hôte.

Lisez le mappage :

```bash
cat /proc/self/uid_map
# "         0          0 4294967295"  => identity map: container UID 0 == host UID 0
# "         0     100000      65536"  => user namespace: container 0 == host 100000
cat /proc/self/gid_map
id
```

Un `uid_map` identité commençant par `0 0` signifie l'absence d'espace de noms utilisateur isolant : vous opérez en tant que véritable root hôte une fois que vous atteignez l'hôte. Une plage mappée signifie qu'un espace de noms utilisateur est actif et que votre root est remappé.

## Pourquoi cela compte pour les autres voies

```bash
# With an identity map, a capability-based escape runs as host root:
#   CAP_SYS_MODULE -> module loads into the host kernel as real root
#   a host disk mount -> files written are owned by real host root
# With a mapped user namespace, the same CAP_* are namespaced:
#   capabilities apply only to resources owned by the mapped range,
#   and host files owned by real root remain unwritable
```

Un espace de noms utilisateur ne vous empêche pas, par exemple, de lire des secrets lisibles par tous, mais il bloque les voies par capacité agissant sur des ressources appartenant à l'hôte, car le noyau vérifie la capacité par rapport à l'espace de noms propriétaire de la ressource.

## Notes d'exploitation

- Vérifiez toujours `uid_map` avant d'investir dans une voie par capacité ou par montage : un mappage identité confirme que la voie aboutit au root hôte ; une plage mappée signifie que vous ne gagnerez peut-être qu'une identité confinée et remappée.
- Docker rootless et Podman utilisent un espace de noms utilisateur par défaut, ce qui explique pourquoi leurs évasions sont plus difficiles et nécessitent souvent un bug conscient des espaces de noms utilisateur plutôt qu'un abus de configuration.
- Certaines évasions de l'hôte (par exemple un montage de socket de runtime qui lance un nouveau conteneur rootful) contournent entièrement la limitation de l'espace de noms utilisateur, car c'est le démon de l'hôte, et non le processus confiné, qui crée le conteneur privilégié.

## Références

- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- [Rootless containers](https://rootlesscontaine.rs/)
- [HackTricks : user namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/user-namespace)
