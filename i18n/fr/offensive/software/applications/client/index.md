---
title: "Client : attaquer le code qui s'exécute sur l'appareil de l'utilisateur"
description: "Techniques offensives contre les logiciels côté client, le code qui s'exécute sur l'appareil de l'utilisateur : interfaces web dans le navigateur, applications mobiles sur Android et iOS, et applications de bureau. L'attaquant contrôle souvent l'environnement d'exécution, si bien que les protections ne sont qu'indicatives et que l'objectif est de retourner le client contre son utilisateur ou de s'en servir pour atteindre le serveur."
keywords:
  - client-side security
  - browser security
  - mobile application security
  - desktop application security
  - XSS
---

# Client

Le logiciel client s'exécute sur l'appareil de l'utilisateur, ce qui inverse l'hypothèse de confiance habituelle : l'attaquant contrôle fréquemment l'environnement dans lequel le code s'exécute, ou peut l'inspecter et l'altérer. Les protections côté client ne sont donc qu'indicatives, et les questions offensives portent sur ce à quoi le client fait confiance, les secrets qu'il détient localement, et la manière dont il peut être retourné contre son propre utilisateur ou utilisé comme tremplin vers le serveur avec lequel il communique.

Ce domaine est organisé **par type de client**, car le modèle d'exécution et les défenses diffèrent nettement : un navigateur applique la politique de même origine (same-origin policy) et un bac à sable (sandbox) autour du code web non fiable, un système d'exploitation mobile confine chaque application et arbitre sa communication interprocessus, et une application de bureau s'exécute avec les propres privilèges de l'utilisateur et une portée locale.

## Types de clients

- **[Navigateur](browser/index.md)** : interfaces web s'exécutant dans le navigateur, où le cross-site scripting, la falsification de requête et le modèle de même origine, CORS et content-security-policy déterminent ce que le script fourni par l'attaquant peut faire.
- **[Mobile](mobile/index.md)** : applications Android et iOS, attaquées via le stockage local non sécurisé, les composants exportés et la communication interprocessus, les liens profonds (deep links), ainsi que le bac à sable de la plateforme et les protections de transport.
- **[Desktop](desktop/index.md)** : applications de bureau installées, attaquées via l'IPC locale et les gestionnaires de protocoles, les canaux de mise à jour non sécurisés, les faiblesses des navigateurs embarqués et d'Electron, et les secrets stockés sur le poste.

## Jonctions

La partie serveur d'une application web, où résident la gestion des entrées, l'authentification et les couches plateforme, runtime et code, se trouve sous [Serveur > Web](../server/web/index.md). Ce domaine ne couvre que la moitié client : ce qui s'exécute dans le navigateur, sur le téléphone ou sur le poste de bureau.

## Références

- [OWASP Web Security Testing Guide: client-side testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/)
- [OWASP Mobile Application Security](https://mas.owasp.org/)
