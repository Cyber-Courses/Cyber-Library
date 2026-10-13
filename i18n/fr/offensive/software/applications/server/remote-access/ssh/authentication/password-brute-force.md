---
title: "Force brute de mot de passe : attaques de mot de passe en ligne contre SSH"
description: "Lorsque SSH autorise l'authentification par mot de passe, il devient une cible de force brute et de password spray. Un succès donne un shell en tant que compte, souvent réutilisable ailleurs. La limitation de débit SSH, les limites de tentatives d'auth par connexion et les configurations à clé seule façonnent l'attaque, favorisant un spraying lent de noms d'utilisateur validés avec des mots de passe contextuels plutôt qu'une force brute rapide."
keywords:
  - force brute ssh
  - password spray
  - hydra
  - maxauthtries
  - réutilisation d'identifiants
---

# Force brute de mot de passe

Lorsque `PasswordAuthentication` est activé, les connexions SSH peuvent être devinées en ligne. L'attaque pratique est façonnée par les défenses de SSH : `MaxAuthTries` limite les tentatives par connexion, `MaxStartups` bride les poignées de main concurrentes, et les outils de type fail2ban bloquent les IP fautives, donc une force brute rapide et large est bruyante et contre-productive. Un spraying lent, un ou quelques mots de passe probables sur une liste d'utilisateurs validés, est l'approche efficace, et tout identifiant valide est un excellent matériau de réutilisation car les comptes SSH sont de vrais comptes système.

```bash
# confirm password auth is offered before spraying
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@<target> 2>&1 | head -1
# spray one password across users (quiet), stop on success, few threads
hydra -L users.txt -p 'Autumn2025!' ssh://<target> -t 4 -f
nxc ssh <target> -u users.txt -p 'Autumn2025!'          # marks shell vs no-shell on hit
# brute one high-value account with a short, targeted list
hydra -l admin -P top-passwords.txt ssh://<target> -t 4 -f
```

## Notes d'exploitation

- Vérifiez d'abord `PasswordAuthentication` ; de nombreux serveurs sont à clé seule (auth par mot de passe désactivée), auquel cas basculez sur les routes par clé plutôt que de deviner.
- Gardez peu de threads et une liste de mots de passe courte et contextuelle : SSH journalise chaque échec, `MaxAuthTries` ferme la connexion après quelques tentatives, et les outils de blocage d'IP réagissent aux rafales.
- Validez d'abord les noms d'utilisateur ([Énumération d'utilisateurs](../enumeration/user-enumeration.md)) pour que les tentatives tombent sur de vrais comptes ; dérivez les mots de passe de l'organisation (saison/année, nom de l'entreprise, identifiants compromis) plutôt que d'une énorme liste générique.
- Tout identifiant SSH fonctionnel est probablement réutilisé ; testez-le immédiatement contre d'autres hôtes et services. NetExec indique si le compte obtient un shell ou est confiné.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec (ssh)](https://github.com/Pennyw0rth/NetExec)

## Références

- [OpenSSH sshd_config: MaxAuthTries, PasswordAuthentication](https://man.openbsd.org/sshd_config)
- [HackTricks: SSH brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
