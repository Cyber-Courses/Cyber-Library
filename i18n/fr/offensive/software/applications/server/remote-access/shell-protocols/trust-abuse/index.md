---
order: 2
title: "Abus de confiance : exploitation de la confiance d'hôte r-command"
description: "Les r-commands authentifient par confiance basée sur l'hôte : .rhosts (par utilisateur) et /etc/hosts.equiv (à l'échelle du système) accordent un accès sans mot de passe aux paires hôte/utilisateur listées, vérifiées par adresse source. Les attaques consistent à implanter une entrée .rhosts, à abuser d'une confiance permissive ou avec joker, à exploiter l'ampleur de hosts.equiv, et à usurper une adresse IP source de confiance, toutes donnant un accès sans mot de passe via rsh, rlogin et rexec."
keywords:
  - abus de confiance
  - rhosts
  - hosts.equiv
  - joker
  - usurpation ip
---

# Abus de confiance

La faiblesse caractéristique des r-commands est leur confiance basée sur l'hôte : `~/.rhosts` (par utilisateur) et `/etc/hosts.equiv` (à l'échelle du système) listent des paires `host user` qui bénéficient d'un accès sans mot de passe, et la vérification se fait contre l'adresse source qui se connecte. C'est une authentification par identité réseau usurpable, et elle est abusée de quatre façons qui s'appliquent uniformément à rsh, rlogin et rexec : implanter une entrée `.rhosts` là où un répertoire personnel est accessible en écriture, exploiter une confiance permissive ou avec joker déjà présente, tirer parti de l'ampleur de `hosts.equiv` (tous les utilisateurs non-root d'un coup), et usurper une IP source de confiance pour satisfaire la vérification sans contrôler l'hôte de confiance.

```bash
# read existing trust where files are accessible (reveals who is trusted)
cat ~*/.rhosts /etc/hosts.equiv 2>/dev/null
# any of the techniques below yields passwordless rsh/rlogin/rexec
```

## Sous-thèmes

- **[Écriture rhosts](rhosts-write.md)** : implantation d'une entrée .rhosts de confiance.
- **[Confiance avec joker](wildcard-trust.md)** : abus de `+ +` et des entrées trop larges.
- **[Confiance hosts.equiv](hosts-equiv-trust.md)** : exploitation de la confiance à l'échelle du système.
- **[Usurpation IP](ip-spoofing.md)** : usurpation d'une adresse source de confiance.

## Références

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
