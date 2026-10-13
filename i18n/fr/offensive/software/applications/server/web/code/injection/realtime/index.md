---
order: 11
title: "Injection temps réel"
description: "Des canaux à longue durée de vie où les corps et métadonnées des messages pilotent le routage, l'autorisation et les sinks, de sorte que chaque trame transporte une injection au-delà de la frontière de la requête HTTP."
keywords:
  - injection temps réel
  - WebSocket
  - server-sent events
  - long polling
  - injection de message
---

# Temps réel

Les transports temps réel maintiennent un canal ouvert bien après la requête HTTP initiale, et sur ce canal l'application échange de nombreux petits messages au lieu de paires requête/réponse distinctes. Les trames WebSocket, les Server-Sent Events et les cycles de long polling partagent tous une propriété qui compte pour un attaquant : le corps et les métadonnées de chaque message pilotent le routage, les décisions d'autorisation et les sinks en aval, pourtant la machinerie qui inspecte une requête HTTP normale ne s'exécute souvent qu'une seule fois, à l'établissement.

Cet écart est la cible. Une trame qui atteint un constructeur de requêtes, un tampon de diffusion, un lanceur de commandes ou un sérialiseur de flux est une entrée contrôlée par l'attaquant qui échappe fréquemment aux règles du WAF, aux middlewares d'authentification et à la validation appliqués aux routes REST. L'autorisation par message est couramment plus faible que l'autorisation par requête, les origines sont souvent non vérifiées lors du handshake, et les formats de flux délimités par des sauts de ligne invitent à la contrefaçon d'enregistrements.

Ce sous-arbre est organisé par transport : traitement des messages WebSocket, Server-Sent Events et long polling.

## Sous-thèmes

- **[WebSocket](websocket/index.md)** : abus du traitement des messages WebSocket après l'upgrade HTTP, où chaque trame pilote l'autorisation, le routage et les sinks côté serveur.

## Pages

- **[Long polling](long-polling.md)** : endpoints maintenus ouverts et de polling où les notifications JSON, les paramètres de requête du poll et les jetons de session dans l'URL du poll atteignent les requêtes et l'autorisation côté serveur.
- **[Server-Sent Events](server-sent-events.md)** : injection de saut de ligne dans un text/event-stream où les lignes data, event et id construites par le serveur proviennent d'un état non fiable, contrefaisant des événements et empoisonnant des flux partagés.

## Outils

- **Burp Suite** : historique WebSockets et repeater pour intercepter et rejouer les trames.
- **websocat** : client WebSocket en ligne de commande pour confectionner des messages bruts.
- **curl -N** : diffuser un endpoint Server-Sent Events pour inspecter les octets non parsés.

## Références

- PortSwigger Web Security Academy: WebSocket security vulnerabilities
- OWASP Web Security Testing Guide: Testing WebSockets
