---
title: "Protocoles shell : attaquer les r-commands de Berkeley"
order: 4
description: "Les r-commands de Berkeley, rsh (514), rlogin (513) et rexec (512), fournissent shell distant, connexion et exécution de commandes sans chiffrement et avec un modèle de confiance basé sur l'hôte. Leurs faiblesses sont la confiance .rhosts et hosts.equiv qui authentifie par adresse source (usurpable), les identifiants et sessions en clair, et l'exécution de commandes via des relations d'hôtes de confiance."
keywords:
  - r-commands
  - rsh
  - rlogin
  - rexec
  - rhosts
---

# Protocoles shell

Les r-commands de Berkeley sont une famille de services hérités d'accès à distance Unix : `rsh` (shell distant, TCP 514), `rlogin` (connexion distante, TCP 513) et `rexec` (exécution distante, TCP 512). Ils sont antérieurs à SSH et partagent deux faiblesses fatales. D'abord, aucun chiffrement : identifiants, commandes et sessions circulent en clair. Ensuite, un modèle de confiance basé sur l'hôte, des fichiers `.rhosts` et `/etc/hosts.equiv` qui accordent un accès sans mot de passe en fonction de l'IP source et du nom d'utilisateur du client, ce qui est une authentification par identité réseau usurpable. La surface d'attaque est donc l'abus de confiance (forger ou implanter la confiance pour obtenir des shells sans mot de passe), l'interception en clair et les chemins d'exécution par commande.

```bash
nmap -p512,513,514 -sV <target>                # rexec, rlogin, rsh
# the services are often found together on legacy Unix
```

## Sous-thèmes

- **[rsh](rsh/index.md)** : shell distant sur le port 514 et son abus de confiance.
- **[rlogin](rlogin/index.md)** : connexion distante sur le port 513.
- **[rexec](rexec/index.md)** : exécution distante sur le port 512.
- **[Abus de confiance](trust-abuse/index.md)** : exploitation de .rhosts et hosts.equiv à travers les r-commands.
- **[Interception de trafic](traffic-interception/index.md)** : capture et rejeu en clair.

## Références

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rexec/rlogin/rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
- [Unix r-commands and .rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
