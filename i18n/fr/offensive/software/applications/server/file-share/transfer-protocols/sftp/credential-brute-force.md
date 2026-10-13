---
order: 1
title: "Brute force d'identifiants : attaques par mot de passe via SSH pour SFTP"
description: "Les comptes SFTP s'authentifient via SSH, le brute force et le spraying de mots de passe visent donc le service SSH. Des identifiants valides donnent un accès aux fichiers par SFTP et, lorsque le compte dispose aussi d'un shell, une exécution de commandes directe. Les limitations de débit SSH et les configurations par clé uniquement façonnent l'attaque, favorisant un spraying lent et discret de noms d'utilisateur validés avec des mots de passe probables."
keywords:
  - brute force ssh
  - password spray
  - sftp
  - hydra
  - réutilisation d'identifiants
---

# Brute force d'identifiants

Parce que l'authentification SFTP est l'authentification SSH, les attaques par mot de passe visent le service SSH sur le port 22. Un mot de passe valide donne un accès aux fichiers par SFTP et, si le compte n'est pas restreint au sous-système SFTP, il donne un shell complet, de sorte que la devinette d'identifiants peut aboutir directement à de l'exécution de commandes. Les serveurs SSH limitent souvent le débit des tentatives et peuvent être en mode clé uniquement (pas d'authentification par mot de passe), l'attaque se mène donc idéalement comme un spraying lent et discret de noms d'utilisateur validés contre des mots de passe probables plutôt qu'un brute force rapide.

```bash
# confirm password auth is even offered before spraying
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@<target>
# spray one likely password across validated users (quiet), stop on success
hydra -L users.txt -p 'Summer2025!' ssh://<target> -t 4 -f
nxc ssh <target> -u users.txt -p passwords.txt          # netexec, marks shell vs no-shell
# a single credential: test both subsystem and shell
sftp user@<target>; ssh user@<target> id
```

## Notes d'exploitation

- Vérifiez d'abord que l'authentification par mot de passe est activée ; de nombreux serveurs SSH sont en mode clé uniquement, auquel cas passez aux voies par [clé](weak-and-stolen-ssh-keys.md) plutôt que de deviner des mots de passe.
- Faites du spraying lentement et avec peu de threads : SSH journalise chaque échec et peut limiter le débit ou bloquer, de sorte qu'une liste d'utilisateurs validés et une courte liste de mots de passe déduite du contexte valent mieux qu'une grande wordlist.
- NetExec indique si un identifiant fonctionnel donne un shell ou seulement le sous-système SFTP ; un shell est une exécution immédiate, un compte limité à SFTP oriente vers l'[évasion de shell restreint](restricted-shell-and-chroot-escape.md).
- Les identifiants SSH sont un matériau de réutilisation de premier choix ; testez toute trouvaille contre d'autres hôtes et services.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec (ssh)](https://github.com/Pennyw0rth/NetExec)

## Références

- [HackTricks: SSH brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [OpenSSH sshd_config auth options](https://man.openbsd.org/sshd_config)
