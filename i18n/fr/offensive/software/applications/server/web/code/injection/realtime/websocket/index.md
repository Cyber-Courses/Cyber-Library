---
order: 1
title: "Injection WebSocket"
description: "Abus du traitement des messages WebSocket après l'upgrade HTTP, où chaque trame pilote l'autorisation, le routage et les sinks côté serveur."
keywords:
  - injection WebSocket
  - trame WebSocket
  - wss
  - traitement des messages
  - abus temps réel
---

# WebSocket

Un WebSocket commence sa vie comme une requête HTTP avec un en-tête `Upgrade: websocket`, mais une fois que le serveur répond `101 Switching Protocols` la connexion quitte entièrement le modèle requête/réponse. Dès lors les deux parties poussent des trames à volonté, et le serveur dispatche chaque trame à travers un handler de messages qui décide de sa signification et de ce qu'elle touche.

C'est dans ce handler que vit la surface d'attaque intéressante. Le handshake a pu être authentifié et vérifié par origine, pourtant chaque trame qui le suit est souvent considérée comme fiable implicitement : le serveur lit une `action`, un `id` ou une `room` dans la trame et agit, sans réexécuter les vérifications qui gardaient l'upgrade. Les corps de trame affluent aussi vers les mêmes backends que n'importe quelle autre entrée, donc une trame peut transporter un fragment SQL, un payload de script ou une expression de template directement jusqu'à un sink tout en contournant le filtrage de la couche HTTP qui l'aurait attrapé sur une route REST.

Cette section couvre les lacunes d'autorisation par message, le cross-site WebSocket hijacking et l'injection de message dans les sinks en aval.

## Pages

- **[Cross-site WebSocket hijacking](cross-site-websocket-hijacking.md)** : un handshake WebSocket qui s'appuie sur les cookies ambiants et saute la validation d'Origin laisse une page attaquante ouvrir un socket authentifié au nom de la victime.
- **[Message injection to sinks](message-injection-to-sinks.md)** : contenu de trame WebSocket affluant sans assainissement vers des sinks SQL, NoSQL, de diffusion XSS stockée, et de commande ou de template, transporté au-delà du filtrage de la couche HTTP.
- **[Per-message authorization](per-message-authorization.md)** : connexions WebSocket autorisées uniquement au handshake, de sorte que chaque trame est considérée comme fiable et peut agir en tant qu'autres utilisateurs ou invoquer des actions privilégiées.

## Outils

- **Burp Suite** : historique WebSockets et repeater pour intercepter, éditer et rejouer les trames.
- **wsrepl** : REPL WebSocket interactif conçu pour les tests d'intrusion.
- **websocat** : client WebSocket en ligne de commande pour scripter et fuzzer les trames brutes.

## Références

- PortSwigger Web Security Academy: WebSocket security vulnerabilities
- OWASP Web Security Testing Guide: Testing WebSockets
