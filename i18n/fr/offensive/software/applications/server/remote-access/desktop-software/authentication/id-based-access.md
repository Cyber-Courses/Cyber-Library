---
title: "Accès par ID : énumérer et cibler les identifiants d'équipement"
description: "Les outils de bureau à distance adressent chaque machine par un identifiant d'équipement numérique, et la connexion nécessite l'ID plus un mot de passe. Comme les ID sont alloués depuis un espace prévisible, un attaquant énumère les ID valides et les associe à de la devinette de mots de passe ou à des mots de passe volés. Historiquement, une liaison faible entre ID et session et des ID prévisibles rendaient pratique la force brute sur la combinaison ID plus mot de passe."
keywords:
  - device id
  - teamviewer id
  - anydesk id
  - id enumeration
  - brute force
---

# Accès par ID

Des produits comme TeamViewer et AnyDesk identifient chaque installation par un ID numérique, et une connexion à distance s'établit en fournissant cet ID et le mot de passe correspondant. L'ID est l'adressage, et ses propriétés façonnent l'attaque : les ID sont tirés d'un espace borné, souvent plus ou moins séquentiel, de sorte que les installations valides peuvent être énumérées, et un attaquant associe un ID découvert à de la devinette de mots de passe ou à un mot de passe volé ou fuité pour se connecter. La robustesse de la liaison entre l'ID et le mot de passe de session compte : là où elle était faible ou l'espace d'ID prévisible, la force brute sur la combinaison ID et mot de passe à grande échelle était pratique, et des campagnes de masse ont abusé de mots de passe réutilisés à travers des ID énumérés.

```bash
# IDs are numeric and enumerable; connections pair ID + password
# an attacker sweeps candidate IDs and attempts known/weak passwords against each
#   (automated against the client/relay connection flow)
# combine with leaked ID:password pairs (from breaches/stealer logs) for direct connects
```

## Notes d'exploitation

- L'ID est de l'adressage, pas un secret ; la sécurité repose sur le mot de passe, donc la véritable attaque est l'énumération d'ID plus une source de mots de passe (devinette, réutilisation ou paires fuitées), voir [mots de passe faibles](weak-passwords.md).
- Les logs d'infostealers et les données de fuites contiennent fréquemment des paires ID et mot de passe pour ces outils, faisant de nombreuses « connexions » des rejeux directs plutôt que de la force brute.
- Une liaison ID-session et une limitation de débit historiquement faibles ont permis aux attaquants de forcer la combinaison par force brute ; les fournisseurs ont resserré cela, donc la faisabilité actuelle dépend de la version du produit et de ses protections.
- Une session connectée équivaut à un contrôle interactif de la machine distante en tant qu'utilisateur connecté ; les configurations d'accès sans surveillance rendent cela atteignable à tout moment, voir [accès sans surveillance](unattended-access.md).

## Références

- [TeamViewer ID and connection model](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
