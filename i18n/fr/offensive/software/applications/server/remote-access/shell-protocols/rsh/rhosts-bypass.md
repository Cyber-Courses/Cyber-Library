---
order: 4
title: "Contournement rhosts : rsh sans mot de passe via la confiance .rhosts"
description: "Les r-commands accordent un accès sans mot de passe lorsque le fichier ~/.rhosts de l'utilisateur cible liste l'hôte et l'utilisateur qui se connectent comme de confiance. Un attaquant qui peut écrire ce fichier, via une autre vulnérabilité, un home accessible en écriture via NFS, ou un point d'ancrage existant, ajoute une entrée de confiance puis se connecte en rsh/rlogin en tant que cet utilisateur sans mot de passe, ce qui est aussi une porte dérobée durable."
keywords:
  - rhosts
  - passwordless
  - trusted host
  - rsh
  - backdoor
---

# Contournement rhosts

Chaque utilisateur peut accorder un accès r-command sans mot de passe en listant des paires `host user` de confiance dans son fichier `~/.rhosts` ; une connexion correspondante est acceptée sans mot de passe. L'usage offensif est de créer ou d'étendre cette confiance. Si un attaquant peut écrire dans le `.rhosts` d'un utilisateur cible, via une vulnérabilité d'écriture de fichier, un répertoire home accessible en écriture (couramment via NFS), ou un point d'ancrage existant en tant qu'un autre utilisateur, il ajoute une entrée faisant confiance à son propre hôte et nom d'utilisateur, puis se connecte en `rsh`/`rlogin` en tant que cible sans identifiant. L'entrée joker `+ +` fait confiance à tout hôte et tout utilisateur, transformant le compte en porte ouverte. Cela sert aussi de persistance.

```bash
# plant trust in the target user's .rhosts (requires a write to their home)
echo '+ +' >> /mnt/nfs-home/victim/.rhosts            # trust everyone (open door)
echo 'attacker-host attacker-user' >> ~victim/.rhosts  # trust a specific identity
# then log in / run commands with no password
rlogin -l victim <target>
rsh -l victim <target> id
```

## Notes d'exploitation

- La condition préalable est une écriture dans le `.rhosts` de la cible ; associez-la à toute primitive d'écriture dans le répertoire home (une [écriture par usurpation d'UID NFS](../../../file-share/network-file-systems/nfs/uid-and-gid-spoofing.md), un partage accessible en écriture, ou un point d'ancrage), ce qui est précisément pourquoi les répertoires home accessibles en écriture sont dangereux avec les r-commands activées.
- `+ +` est l'entrée maximale (tout hôte, tout utilisateur) ; une entrée ciblée `host user` est plus furtive.
- C'est à la fois accès et persistance : la confiance plantée survit et accorde une ré-entrée sans mot de passe jusqu'à ce que le fichier soit nettoyé.
- Le même mécanisme `.rhosts` sert rsh, rlogin et rexec ; voir [écriture rhosts](../trust-abuse/rhosts-write.md) pour la technique d'écriture dans la vue abus de confiance.

## Références

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rsh/.rhosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
