---
title: "Abus de session : détournement, shadowing et redirection de périphériques"
description: "Au-delà de la connexion, les sessions RDP elles-mêmes sont détournables. Un administrateur ou un attaquant de niveau SYSTEM sur un hôte prend le contrôle des sessions déconnectées ou actives d'autres utilisateurs sans leur mot de passe, fait du shadowing pour les observer en direct, et les canaux de redirection de périphériques (presse-papiers, lecteurs) laissent fuiter des données entre client et serveur. Ces techniques transforment un accès à l'hôte en capture latérale d'identifiants et de données."
keywords:
  - rdp session
  - session hijacking
  - shadowing
  - clipboard
  - drive redirection
---

# Abus de session

Les sessions RDP sont une surface d'attaque à part entière, exploitée une fois qu'un attaquant dispose déjà d'un accès privilégié à un serveur de terminaux ou à un hôte multi-utilisateurs. Windows permet à un compte suffisamment privilégié de se connecter à la session d'un autre utilisateur : le détournement prend le contrôle d'une session déconnectée ou même active sans le mot de passe de la victime, et le shadowing observe une session en direct. Séparément, les canaux virtuels de redirection de périphériques de RDP, presse-papiers et mappage de lecteurs, déplacent des données entre le client et le serveur et les laissent fuiter vers quiconque contrôle l'une des extrémités. Ces techniques convertissent un accès à l'hôte en capture des sessions, identifiants et données des autres utilisateurs.

```bash
# enumerate sessions on a host you have privileged access to
query user            # or: qwinsta   (session IDs, states: Active/Disc)
```

## Sous-thèmes

- **[Détournement de session](session-hijacking.md)** : prendre le contrôle de la session d'un autre utilisateur sans son mot de passe.
- **[Shadowing de session](session-shadowing.md)** : observer une session en direct.
- **[Redirection du presse-papiers](clipboard-redirection.md)** : capturer des données via le canal presse-papiers.
- **[Redirection de lecteurs](drive-redirection.md)** : atteindre les lecteurs client mappés.

## Références

- [Microsoft: Remote Desktop Services sessions](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP session abuse](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
