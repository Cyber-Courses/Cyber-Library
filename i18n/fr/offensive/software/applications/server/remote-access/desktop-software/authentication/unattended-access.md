---
title: "Accès sans surveillance : abuser d'un accès de bureau à distance permanent"
description: "L'accès sans surveillance permet de se connecter à un outil de bureau à distance à tout moment sans utilisateur présent, authentifié par un mot de passe sans surveillance fixe. Lorsque ce mot de passe est faible, par défaut, réutilisé ou récupéré depuis la configuration stockée du client, un attaquant se connecte à la machine quand il le souhaite, obtenant un contrôle interactif persistant sans que personne ne le sache."
keywords:
  - unattended access
  - unattended password
  - persistent access
  - teamviewer
  - anydesk
---

# Accès sans surveillance

L'accès sans surveillance est la fonctionnalité qui fait de ces outils une colonne vertébrale de l'administration à distance : il permet de se connecter à une machine à tout moment, sans utilisateur présent pour approuver ou lire à voix haute un mot de passe de session, authentifié à la place par un mot de passe d'accès sans surveillance fixe configuré sur le client. Pour un attaquant, c'est un contrôle permanent et persistant s'il obtient ce mot de passe, ce qu'il fait en le devinant (souvent faible ou réutilisé), en le trouvant réglé sur une valeur par défaut ou en le récupérant depuis la configuration stockée du client sur une machine à laquelle il a déjà touché. Une fois détenu, l'attaquant se connecte quand il le choisit et contrôle la machine de façon interactive, et c'est aussi un mécanisme de persistance furtif.

```bash
# the unattended password is stored client-side; recover it where you have local access
#   TeamViewer/AnyDesk store configuration (and in some versions recoverable secrets)
#   in the registry / app data; dump and decrypt per the product's storage
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>/dev/null       # (example location)
# with the unattended ID + password, connect at will from the attacker client
```

## Notes d'exploitation

- L'accès sans surveillance transforme l'outil en porte dérobée durable : un mot de passe sans surveillance récupéré ou faible accorde une connexion à tout moment, c'est donc à la fois de l'accès et de la persistance, plus discret que de déployer un nouvel outillage.
- Le mot de passe sans surveillance est stocké sur le client ; là où vous avez déjà un accès local (même bref), le récupérer depuis la configuration/le registre du produit donne une ré-entrée distante durable, parfois récupérable sous une forme équivalente au texte clair selon la version.
- Les mots de passe sans surveillance faibles ou par défaut sont devinables directement ; la réutilisation signifie qu'un mot de passe trouvé ailleurs peut être celui sans surveillance.
- C'est le résultat d'authentification à plus forte valeur car il s'agit d'un accès permanent ; combinez avec [mots de passe faibles](weak-passwords.md) et [accès par ID](id-based-access.md) pour y parvenir.

## Références

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [AnyDesk unattended access](https://anydesk.com/en/)
