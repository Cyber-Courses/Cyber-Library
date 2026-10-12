---
title: "Authentification : attaquer le contrôle d'accès des logiciels de bureau à distance"
description: "Les outils tiers de bureau à distance s'authentifient par un identifiant d'équipement plus un mot de passe de session ou d'accès sans surveillance. Les faiblesses sont des ID et des mots de passe devinables ou forçables par force brute, des configurations d'accès sans surveillance qui autorisent une connexion sans mot de passe ou à mot de passe faible à tout moment, et des mots de passe réutilisés ou par défaut. Obtenir l'accès donne un contrôle interactif de la machine distante en tant qu'utilisateur connecté."
keywords:
  - id-based access
  - unattended access
  - weak password
  - teamviewer
  - anydesk
---

# Authentification

Ces produits authentifient une connexion avec un identifiant d'équipement (l'ID TeamViewer/AnyDesk) et un mot de passe, soit un mot de passe de session à usage unique montré à l'utilisateur présent, soit un mot de passe d'accès sans surveillance configuré pour se connecter sans personne au clavier. La surface d'attaque en découle. Les ID sont numériques et, combinés à des mots de passe faibles, forçables par force brute. L'accès sans surveillance, mis en place par commodité, permet à un attaquant de se connecter à tout moment, et là où son mot de passe est faible, par défaut ou absent, cela équivaut à un contrôle à distance permanent. Et les mots de passe sont réutilisés et faibles comme tous les autres. Le succès, c'est un contrôle interactif complet de la machine distante en tant qu'utilisateur connecté, souvent avec les privilèges que détient cet utilisateur.

## Sous-sujets

- **[Accès par ID](id-based-access.md)** : énumérer et cibler les identifiants d'équipement.
- **[Accès sans surveillance](unattended-access.md)** : abuser d'un accès permanent sans mot de passe ou à mot de passe faible.
- **[Mots de passe faibles](weak-passwords.md)** : deviner les mots de passe de session et sans surveillance.

## Références

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
