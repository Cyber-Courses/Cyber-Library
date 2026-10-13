---
order: 2
title: "Confiance avec joker : abus de + + et des entrées r-command trop larges"
description: "Un jeton + dans .rhosts ou hosts.equiv est un joker correspondant à n'importe quel hôte ou n'importe quel utilisateur, donc une entrée comme + + fait confiance à tout le monde pour un accès sans mot de passe. Les entrées trop larges, un + seul, un hôte avec joker, ou un netgroup qui se résout largement, permettent à un attaquant depuis n'importe quelle adresse (ou une adresse usurpée) de se connecter sans mot de passe."
keywords:
  - joker
  - + +
  - rhosts
  - hosts.equiv
  - confiance ouverte
---

# Confiance avec joker

Les fichiers de confiance des r-commands traitent `+` comme un joker : `+` dans le champ hôte correspond à n'importe quel hôte, `+` dans le champ utilisateur correspond à n'importe quel utilisateur, et la tristement célèbre entrée `+ +` fait confiance à chaque hôte et chaque utilisateur pour un accès sans mot de passe. La confiance trop large est une mauvaise configuration courante dans le monde réel : un `+` seul, une entrée d'hôte avec joker, ou un netgroup qui se résout en un ensemble large. Lorsqu'une telle entrée existe dans le `.rhosts` d'un utilisateur ou dans `/etc/hosts.equiv`, un attaquant depuis n'importe quelle adresse source (même pas besoin d'usurpation) se connecte ou exécute des commandes en tant que compte de confiance sans mot de passe.

```bash
# detect over-broad trust where the files are readable
grep -R '+' /etc/hosts.equiv ~*/.rhosts 2>/dev/null     # a bare + or "+ +" is wide open
# where present, access needs no password and often no spoofing
rlogin -l <user> <target>
rsh -l <user> <target> id
```

## Notes d'exploitation

- `+ +` et un `+` seul sont les cas maximaux : ils font confiance à tout le monde, donc tout attaquant atteignable obtient un accès sans mot de passe, sans contrôle d'hôte de confiance ni usurpation.
- Un `+` uniquement dans le champ utilisateur d'une entrée d'hôte fait confiance à tous les utilisateurs de cet hôte ; dans le champ hôte, à tous les hôtes pour cet utilisateur ; lisez les champs attentivement pour voir l'ampleur de la confiance.
- Les entrées netgroup (`+@group`) et les entrées d'hôte avec joker peuvent se résoudre plus largement que prévu ; lorsque vous pouvez lire le fichier, évaluez ce à quoi elles correspondent réellement.
- C'est le cas « déjà ouvert » de l'abus de confiance ; lorsqu'aucun joker n'existe, [implantez-en un](rhosts-write.md) ou [usurpez un hôte de confiance spécifique](ip-spoofing.md).

## Références

- [man 5 rhosts (token semantics)](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
