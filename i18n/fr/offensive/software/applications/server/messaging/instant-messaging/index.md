---
title: "Messagerie instantanée : attaquer les protocoles de chat et les plateformes de chat d'équipe"
order: 2
description: "Attaquer l'infrastructure de messagerie en temps réel : les protocoles de chat ouverts avec de multiples implémentations serveur (IRC, XMPP, Matrix) et les plateformes de chat d'équipe auto-hébergées atteintes comme des applications web et des API (Mattermost, Rocket.Chat)."
keywords:
  - instant messaging
  - chat server
  - IRC
  - XMPP
  - Matrix
  - Mattermost
  - Rocket.Chat
---

# Chat

Les serveurs de messagerie en temps réel se répartissent en deux modèles d'attaque. Les protocoles ouverts (IRC, XMPP, Matrix) sont attaqués sur le fil : vous parlez le protocole pour énumérer le serveur, abuser de ses services et de sa fédération, et exploiter le démon. Les plateformes de chat d'équipe auto-hébergées (Mattermost, Rocket.Chat) sont attaquées comme des applications web : des API REST et websocket où les faiblesses sont le contournement d'authentification, les lacunes d'autorisation, l'injection, et l'exécution de code via intégration ou plugin.

## Triage

```bash
nmap -sV -p6667,6697,5222,5223,5269,8008,8448,8065,3000 <target>
#  6667/6697 IRC   5222/5269 XMPP   8008/8448 Matrix   8065 Mattermost   3000 Rocket.Chat
curl -s http://<target>:8065/api/v4/system/ping        # Mattermost
curl -s http://<target>:3000/api/info                  # Rocket.Chat
curl -s https://<target>/_matrix/client/versions       # Matrix homeserver
```

Un port IRC/XMPP/Matrix mène à [Protocoles](protocols/index.md) ; une application web Mattermost ou Rocket.Chat mène à [Plateformes de chat d'équipe](team-chat-platforms/index.md).

## Sous-thèmes

- **[Protocoles](protocols/index.md)** : IRC, XMPP et Matrix, attaqués sur le fil de l'énumération jusqu'à l'exploitation serveur.
- **[Plateformes de chat d'équipe](team-chat-platforms/index.md)** : Mattermost et Rocket.Chat auto-hébergés, attaqués via leurs API web.

## Références

- [RFC 1459: Internet Relay Chat](https://www.rfc-editor.org/rfc/rfc1459)
- [RFC 6120: XMPP Core](https://www.rfc-editor.org/rfc/rfc6120)
- [Matrix specification](https://spec.matrix.org/latest/)
