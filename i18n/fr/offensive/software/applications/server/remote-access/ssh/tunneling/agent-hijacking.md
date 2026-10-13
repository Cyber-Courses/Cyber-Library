---
order: 4
title: "Détournement d'agent : réutilisation d'un agent SSH transmis"
description: "La transmission d'agent SSH expose la socket d'agent d'un utilisateur sur l'hôte distant pour qu'il puisse s'authentifier plus loin sans copier de clés. Un attaquant qui contrôle, ou a les droits root sur, un hôte où un utilisateur a transmis son agent utilise cette socket active pour s'authentifier en tant que l'utilisateur vers tout système que ses clés déverrouillent, sans jamais posséder la clé privée."
keywords:
  - transmission d'agent
  - ssh-agent
  - SSH_AUTH_SOCK
  - détournement d'agent
  - réutilisation de clé
---

# Détournement d'agent

La transmission d'agent SSH (`ssh -A`) permet à un utilisateur de porter son authentification vers un hôte distant : l'agent, détenant ses clés privées, reste sur son poste de travail, et une socket sur l'hôte distant relaie les requêtes de signature vers lui. Cela évite de copier des clés sur les hôtes intermédiaires, mais cela signifie que tant que l'utilisateur est connecté, tout processus sur l'hôte distant capable d'atteindre la socket d'agent transmise peut lui demander de signer des authentifications, en tant que l'utilisateur, vers tout système que ses clés déverrouillent. Un attaquant contrôlant (ou ayant root sur) un tel hôte détourne la socket active et se déplace plus loin en tant que l'utilisateur, sans jamais voir la clé privée elle-même.

```bash
# on a host where a user has forwarded their agent, find the agent socket
ls -l /tmp/ssh-*/agent.*                        # forwarded agent sockets
# as root (or the user), point SSH at the socket and use the user's keys
SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.1234 ssh-add -l    # list the keys the agent holds
SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.1234 ssh user@next-internal-host
```

## Notes d'exploitation

- La fenêtre est tant que la session de l'utilisateur est active : la socket transmise ne fonctionne que tant que l'agent sur son poste de travail est atteignable, donc le détournement est opportuniste et calé sur les sessions actives.
- root sur l'hôte intermédiaire peut accéder à la socket d'agent transmise de n'importe quel utilisateur sous `/tmp/ssh-*` ; un attaquant non-root a besoin d'un accès à la socket spécifique (même UID ou une erreur de permissions).
- Vous n'obtenez jamais la clé privée, seulement la possibilité de vous authentifier tant que l'agent est atteignable ; utilisez-la immédiatement pour atteindre le prochain saut ou pour implanter un accès durable (une entrée `authorized_keys`) en tant que l'utilisateur.
- C'est pourquoi transmettre un agent vers un hôte non fiable ou multi-locataire est dangereux, et pourquoi les bastions ([Abus d'hôte de rebond](jump-host-abuse.md)) qui reçoivent des agents transmis sont des points de détournement de choix.

## Outils

- [ssh-add / ssh-agent](https://man.openbsd.org/ssh-agent)

## Références

- [OpenSSH agent forwarding](https://man.openbsd.org/ssh#A)
- [HackTricks: SSH agent hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
