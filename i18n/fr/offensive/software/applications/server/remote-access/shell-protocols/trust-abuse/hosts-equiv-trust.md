---
order: 3
title: "Confiance hosts.equiv : exploitation de la confiance r-command à l'échelle du système"
description: "/etc/hosts.equiv accorde un accès r-command sans mot de passe pour les hôtes de confiance à tous les utilisateurs non-root d'un coup, une confiance plus large que les .rhosts par utilisateur. Une entrée trop permissive, un hôte de confiance qu'un attaquant contrôle ou peut usurper, ou un accès en écriture au fichier (après élévation) donnent un accès sans mot de passe en tant que n'importe quel compte non-root du système."
keywords:
  - hosts.equiv
  - à l'échelle du système
  - confiance
  - sans mot de passe
  - tous les utilisateurs
---

# Confiance hosts.equiv

`/etc/hosts.equiv` est le fichier de confiance à l'échelle du système pour les r-commands : les hôtes (et éventuellement les utilisateurs) qui y figurent bénéficient d'un accès sans mot de passe à tous les comptes non-root du système simultanément. Cela le rend plus large et plus dangereux que le `.rhosts` d'un seul utilisateur. Les cas d'abus : le fichier liste déjà un hôte que l'attaquant contrôle ou peut usurper (accès sans mot de passe en tant que n'importe quel utilisateur non-root depuis là), il contient une entrée trop large ou avec joker, ou l'attaquant a obtenu assez de privilèges locaux pour l'écrire et s'accorder une confiance à l'échelle du système. root fait exception : la confiance root ne vit que dans `/.rhosts`.

```bash
# read it to see which hosts are trusted and how broadly
cat /etc/hosts.equiv 2>/dev/null
# from a trusted (or spoofed) host, log in as any non-root user with no password
rlogin -l <anyuser> <target>
rsh -l <anyuser> <target> id
# if writable after local elevation, grant system-wide trust
echo '+' >> /etc/hosts.equiv      # trust all hosts (open door for all non-root users)
```

## Notes d'exploitation

- L'ampleur est l'intérêt : une seule entrée de confiance expose chaque compte non-root, donc la confiance `hosts.equiv` est un gain plus large qu'un [.rhosts](rhosts-write.md) par utilisateur ; énumérez les utilisateurs existants pour choisir un compte cible utile.
- root n'est pas couvert par `hosts.equiv` ; le root sans mot de passe exige `/.rhosts`, notez donc la distinction lorsque vous ciblez le compte root.
- Le fichier appartient à root : l'écrire est donc une étape de post-élévation/persistance ; le vecteur initial est généralement un fichier déjà permissif plus un hôte source de confiance ou [usurpable](ip-spoofing.md).
- S'applique uniformément à rsh, rlogin et rexec sur l'hôte.

## Références

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
