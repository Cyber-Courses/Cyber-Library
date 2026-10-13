---
order: 1
title: "Autorisation par message"
description: "Connexions WebSocket autorisées uniquement au handshake, de sorte que chaque trame est considérée comme fiable et peut agir en tant qu'autres utilisateurs ou invoquer des actions privilégiées."
keywords:
  - autorisation par message
  - autorisation WebSocket
  - IDOR WebSocket
  - broken access control
  - altération de trame
---

# Autorisation par message

De nombreux serveurs WebSocket authentifient et autorisent une seule fois, durant le handshake d'upgrade, puis traitent chaque trame de la connexion résultante comme appartenant à ce principal autorisé. Le handler lit un nom d'action et un ensemble d'identifiants dans chaque trame et agit dessus, en faisant confiance au fait que celui qui possède le socket a le droit de toucher ce que la trame nomme. Cette supposition est l'ouverture : l'identité a été vérifiée, mais l'objet, la room ou l'opération privilégiée spécifique nommée dans chaque trame ne l'est jamais.

## Changer l'identifiant d'objet

Un socket de chat ou de collaboration transporte typiquement un id de document, de conversation ou de ressource dans chaque trame. Si le handler charge cet id sans revérifier que l'utilisateur connecté le possède, incrémenter ou échanger la valeur atteint les données d'autres utilisateurs sur la même connexion authentifiée.

```json
{"action": "message.list", "conversationId": 4021}
```

Rejouez avec un id voisin et lisez une conversation qui appartient à quelqu'un d'autre :

```json
{"action": "message.list", "conversationId": 4022}
```

Le même schéma s'applique aux écritures. Une trame qui publie en tant qu'utilisateur connecté transporte souvent un `senderId` ou un `authorId` auquel le serveur fait confiance verbatim :

```json
{"action": "message.send", "conversationId": 4022, "senderId": 7, "body": "approved, release the funds"}
```

Définir `senderId` sur un autre compte contrefait un message attribué à cet utilisateur, car l'identité du handshake n'est jamais réconciliée avec le champ du corps.

## Changer la room ou le channel

Les serveurs pub/sub associent un socket à des channels via des trames d'abonnement. Si l'abonnement n'est pas autorisé par channel, une connexion à faible privilège peut rejoindre un flux administratif ou privé et recevoir tout ce qui y est diffusé :

```json
{"action": "subscribe", "room": "admin:alerts"}
```

```json
{"action": "subscribe", "room": "tenant:acme:billing"}
```

Une fois abonné, l'attaquant est un destinataire passif de chaque événement que le serveur publie dans cette room, ce qui inclut souvent les enregistrements d'autres tenants dans un déploiement multitenant.

## Invoquer des actions privilégiées

Les serveurs multiplexent fréquemment les opérations ordinaires et administratives sur une seule connexion, filtrant les opérations admin derrière une UI que le client non-admin ne rend jamais. Le socket accepte quand même les trames. Énumérer les noms d'action atteint des opérations que le compte n'était jamais censé appeler :

```json
{"action": "user.setRole", "userId": 7, "role": "admin"}
```

```json
{"action": "config.update", "key": "signups.open", "value": true}
```

Comme la couche par message vérifie seulement que la connexion est authentifiée, et non que ce principal peut exécuter cette action sur cette cible, chacune de ces trames s'exécute avec la même autorité que la console admin légitime.

## Trouver la surface d'actions

Capturez le trafic propre du client pour apprendre le schéma de trame, puis mutez-le. Les outils de développement du navigateur exposent chaque trame sous la vue WS du panneau réseau, et les mêmes trames peuvent être rejouées et fuzzées depuis un client autonome :

```javascript
// run in the authenticated user's browser console; the handshake sends cookies automatically
const ws = new WebSocket('wss://target.example/stream');
ws.onopen = () => {
  for (let id = 4000; id < 4100; id++) {
    ws.send(JSON.stringify({ action: 'message.list', conversationId: id }));
  }
};
ws.onmessage = (e) => console.log(e.data);
```

Depuis un client Node `ws` autonome il n'y a pas de cookie ambiant, donc passez une chaîne de session capturée explicitement : `new WebSocket(url, { headers: { Cookie: 'session=...' } })`. Les trames qui renvoient des données pour des ids hors de l'ensemble propre de l'utilisateur connecté confirment que l'autorisation vit uniquement au handshake.

## Outils

- **Burp Suite** : historique WebSockets et repeater pour rejouer les trames avec des ids, rooms et actions échangés.
- **wsrepl** : REPL WebSocket interactif pour énumérer les noms d'action et les identifiants d'objet.
- **websocat** : client WebSocket en ligne de commande pour passer un cookie capturé et fuzzer les trames.

## Références

- PortSwigger Web Security Academy, "Testing for WebSockets security vulnerabilities"
- OWASP Web Security Testing Guide, "Testing WebSockets"
