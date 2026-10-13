---
order: 3
title: "hosts.equiv : abus de confiance r-command à l'échelle du système"
description: "/etc/hosts.equiv accorde un accès r-command sans mot de passe à l'échelle du système pour les hôtes de confiance listés, s'appliquant à tous les utilisateurs non root. Un hosts.equiv permissif ou modifié par un attaquant, en particulier une entrée joker, permet à un attaquant depuis un hôte de confiance (ou usurpé) de se connecter en tant que n'importe quel utilisateur sans mot de passe, une confiance plus large que le .rhosts par utilisateur."
keywords:
  - hosts.equiv
  - system-wide trust
  - rsh
  - passwordless
  - wildcard
---

# hosts.equiv

`/etc/hosts.equiv` est l'équivalent à l'échelle du système de `.rhosts` : il liste des hôtes (et facultativement des utilisateurs) de confiance pour un accès r-command sans mot de passe, et il s'applique à tous les comptes non root du système en une fois. Cette ampleur le rend plus dangereux que le `.rhosts` d'un seul utilisateur. Un `hosts.equiv` permissif, qui liste un hôte que l'attaquant contrôle ou peut usurper, ou une entrée joker, permet à l'attaquant de se connecter en tant que pratiquement n'importe quel utilisateur depuis cet hôte sans mot de passe. Si l'attaquant peut écrire dans `hosts.equiv` (nécessitant un accès local élevé), il s'accorde directement une confiance à l'échelle du système.

```bash
# if a trusted host is attacker-controlled or spoofable, log in as any user
rlogin -l <anyuser> <target>          # accepted if the source host is in hosts.equiv
rsh -l <anyuser> <target> id
# a wildcard/permissive hosts.equiv (e.g. a bare "+" or a broad host entry) trusts widely
# if writable (post-elevation), grant system-wide trust:
echo '+' >> /etc/hosts.equiv
```

## Notes d'exploitation

- `hosts.equiv` s'applique à tous les utilisateurs non root, donc une seule entrée permissive expose chaque compte de l'hôte ; il ne couvre pas root (root utilise uniquement `/.rhosts`), un point de confusion fréquent.
- Un `+` nu ou une entrée d'hôte trop large est la faiblesse maximale ; combiné à l'[usurpation d'IP](ip-spoofing.md) d'une adresse de confiance, il accorde l'accès sans contrôler l'hôte de confiance.
- Écrire dans `hosts.equiv` nécessite un accès local élevé (il appartient à root), c'est donc plus une étape de persistance/escalade qu'un vecteur initial ; le vecteur initial est généralement un fichier déjà permissif plus une source de confiance ou usurpée.
- Le même fichier régit rsh, rlogin et rexec ; voir la [page hosts.equiv de l'abus de confiance](../trust-abuse/hosts-equiv-trust.md) pour la vue transversale.

## Références

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
