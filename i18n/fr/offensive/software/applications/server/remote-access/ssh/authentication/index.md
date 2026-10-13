---
order: 1
title: "Authentification : attaquer la connexion SSH"
description: "SSH authentifie par mot de passe ou par clé publique, et les deux sont des surfaces d'attaque : les mots de passe par défaut et faibles cèdent à la force brute et au spraying, les clés privées exposées ou réutilisées authentifient sans mot de passe, et les clés issues de générateurs défaillants sont prévisibles. La méthode d'authentification que le serveur propose par utilisateur détermine aussi quelle attaque s'applique."
keywords:
  - authentification ssh
  - mot de passe
  - clé publique
  - identifiants par défaut
  - clés faibles
---

# Authentification

SSH accorde l'accès par mot de passe ou par authentification à clé publique, et la configuration du serveur décide laquelle est proposée. Les deux sont attaquées. Les mots de passe tombent face aux vérifications d'identifiants par défaut, à la force brute et au spraying, limités par l'éventuelle limitation de débit. L'authentification à clé publique, le mécanisme réel le plus courant, est attaquée non pas en cassant la clé mais en trouvant la clé privée, exposée dans des partages, des sauvegardes et des dépôts, ou réutilisée sur plusieurs hôtes, et en exploitant des clés générées assez faiblement pour être prévisibles. Énumérer la méthode que le serveur accepte par utilisateur oriente l'effort.

```bash
# which authentication methods does the server offer for a user?
ssh -o PreferredAuthentications=none -o PubkeyAuthentication=no user@<target> 2>&1 | grep -i 'authentications that can continue'
```

## Sous-thèmes

- **[Identifiants par défaut](default-credentials.md)** : connexions SSH d'usine et constructeur.
- **[Force brute de mot de passe](password-brute-force.md)** : attaques de mot de passe en ligne et spraying.
- **[Exposition de clé publique](public-key-exposure.md)** : recherche et réutilisation de clés privées.
- **[Génération de clés faibles](weak-key-generation.md)** : clés prévisibles issues de générateurs défaillants.

## Références

- [OpenSSH sshd_config authentication options](https://man.openbsd.org/sshd_config)
- [HackTricks: SSH authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
