---
title: "Navigateur : attaquer les interfaces web côté navigateur"
description: "Techniques offensives contre le code web exécuté dans le navigateur : le cross-site scripting et ses contextes d'exécution, le cross-site request forgery, le clickjacking et la redirection d'interface, ainsi que le contournement de la same-origin policy, de CORS, de PostMessage, des WebSockets et de la Content Security Policy censés contenir le code web non fiable."
keywords:
  - cross-site scripting
  - CSRF
  - same-origin policy
  - CORS
  - content security policy
---

# Navigateur

Le navigateur exécute du code non fiable provenant de tous les sites à la fois et est censé maintenir chaque origine isolée des autres et des données de l'utilisateur. Les attaques web côté client vivent dans les failles de ce modèle : faire exécuter un script attaquant dans une origine de confiance, faire agir le navigateur de la victime contre sa volonté, et abuser des mécanismes de messagerie et de partage de ressources entre origines qui assouplissent volontairement l'isolation.

## La surface

- **Cross-site scripting** : exécuter un script attaquant dans une origine de confiance (réfléchi, stocké et DOM-based), ses contextes d'exécution et ses points d'entrée (sinks), ainsi que les contournements de filtres et de sanitizers qui les atteignent.
- **Falsification de requêtes et redirection d'interface** : le cross-site request forgery, le clickjacking, et d'autres façons de faire exécuter des actions par le navigateur authentifié de la victime.
- **Le modèle d'origine** : la same-origin policy et ses limites, les mauvaises configurations CORS, `PostMessage` et la messagerie inter-documents, les WebSockets, et les fuites inter-sites.
- **Confinement** : contourner ou affaiblir la Content Security Policy et d'autres défenses côté client, et abuser du stockage et des extensions du navigateur.

## Jonctions

L'injection côté serveur, l'authentification et les failles logiques, y compris les cas où le serveur d'une application web réfléchit ou stocke les données qui deviennent une charge utile XSS, relèvent de [Server > Web](../../server/web/index.md) ; ce domaine couvre l'exécution côté navigateur et le modèle d'origine.

## Références

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP WSTG: client-side testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/)
