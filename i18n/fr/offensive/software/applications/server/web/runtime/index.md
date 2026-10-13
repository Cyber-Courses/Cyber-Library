---
order: 2
title: "Runtime : attaquer le moteur de langage derrière une application web"
description: "Techniques offensives contre l'interpréteur ou la VM qui exécute le code d'une application web : désérialisation non sécurisée, wrappers de flux du moteur, et évasion des sandboxes et restrictions de fonctions du runtime."
keywords:
  - exploitation du runtime
  - désérialisation non sécurisée
  - wrappers php
  - contournement de disable_functions
  - évasion de sandbox
---

# Runtime

Le **runtime** est le moteur de langage qui exécute l'application : le processus PHP, Python, Node.js, Java, Ruby ou .NET qui analyse et exécute le code des développeurs. Cette couche possède une classe de vulnérabilités distincte, séparée de la logique applicative au-dessus ([Code](../code/index.md)) et de la plateforme serveur en dessous (Platform). Le test pour savoir si un bug relève d'ici : *existerait-il encore si le code source de l'application était bénin et si le serveur frontal était durci, parce que la faiblesse réside dans la façon dont le moteur analyse, exécute ou fait le pont vers l'OS ?*

Les bugs de runtime sont massivement **propres au moteur** : une chaîne d'injection d'objet PHP, une chaîne de gadgets Java et un payload `pickle` Python partagent un nom et rien d'autre. Donc ce domaine est organisé par primitive, puis par langage, parce que les gadgets et les payloads divergent par moteur.

## Où ça casse

- **Désérialisation.** Les moteurs qui reconstruisent des objets à partir d'un flux d'octets exécutent des callbacks définis par le moteur (méthodes magiques, `__reduce__`, `readObject`) pendant la reconstruction. Alimentez le sink avec des octets contrôlés par l'attaquant et ces callbacks deviennent une chaîne d'exécution de code.
- **Wrappers de flux.** Les moteurs exposent des pseudo-protocoles (`php://`, `phar://`, `data://`) qui transforment une opération de fichier anodine en divulgation de source, inclusion distante ou désérialisation.
- **Sandboxes et restrictions de fonctions.** Les déploiements qui tentent de contenir le runtime (`disable_functions`, `open_basedir`, un sandbox de langage, un `vm` Node) reposent sur des frontières que le moteur lui-même peut être amené à franchir.

## Sections

- **[Désérialisation non sécurisée](insecure-deserialization/index.md)** : reconstruire des objets contrôlés par l'attaquant en RCE, par langage (PHP, Python, Java, Ruby, .NET, Node.js).
- **[Wrappers de fichiers et gestionnaires de flux](file-wrappers-and-stream-handlers/index.md)** : les wrappers de flux du moteur qui escaladent les opérations de fichier vers la divulgation, la désérialisation et la RCE.
- **[Évasion de sandbox et de restriction de fonctions](sandbox-and-function-restriction-escape/index.md)** : déjouer `disable_functions`, `open_basedir`, et les sandboxes de langage ou de VM.

## Références

- PortSwigger Web Security Academy: Insecure deserialization
- OWASP: Deserialization of untrusted data
