---
order: 5
title: "rsh : attaquer le service de shell distant"
description: "rsh (rshd sur TCP 514) exécute une commande ou ouvre un shell sur un hôte distant, en s'authentifiant par la confiance basée sur l'hôte dans .rhosts et /etc/hosts.equiv plutôt que par un mot de passe. Un attaquant abuse de cette confiance, en écrivant un fichier .rhosts, en exploitant un hosts.equiv permissif, ou en usurpant une IP source de confiance, pour exécuter des commandes sans identifiant, et intercepte la session en clair."
keywords:
  - rsh
  - rshd
  - port 514
  - rhosts
  - remote shell
---

# rsh

`rsh` exécute une commande (ou un shell interactif) sur un hôte distant via `rshd` sur TCP 514. Son authentification est basée sur l'hôte : le serveur vérifie si l'IP source et le nom d'utilisateur qui se connectent sont de confiance via le `~/.rhosts` de l'utilisateur cible ou le `/etc/hosts.equiv` à l'échelle du système, et si oui, accorde l'accès sans mot de passe. Cette conception est toute la vulnérabilité. Un attaquant qui peut amener le serveur à lui faire confiance, en écrivant une entrée `.rhosts`, en abusant d'un `hosts.equiv` permissif, ou en usurpant une adresse source de confiance, exécute des commandes en tant qu'utilisateur cible sans identifiant, et parce que le canal est en clair, toute session réelle est aussi interceptable.

```bash
# if trusted (or trust is permissive), run commands with no password
rsh -l <user> <target> id
rsh -l root <target> 'cat /etc/shadow'
# test reachability
nmap -p514 -sV <target>
```

## Sous-thèmes

- **[Contournement rhosts](rhosts-bypass.md)** : accès sans mot de passe via la confiance .rhosts.
- **[hosts.equiv](hosts-equiv.md)** : abus de confiance à l'échelle du système.
- **[Usurpation d'IP](ip-spoofing.md)** : usurper une adresse source de confiance.
- **[Interception en clair](cleartext-interception.md)** : capturer la session non chiffrée.

## Références

- [rshd manual](https://linux.die.net/man/8/rshd)
- [HackTricks: rsh (514)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
