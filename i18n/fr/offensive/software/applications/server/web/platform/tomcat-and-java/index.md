---
order: 5
title: "Serveurs d'applications Tomcat et Java : AJP, applications de gestion et traitement des chemins"
description: "Mauvaises configurations de plateforme dans Tomcat et les serveurs d'applications Java : lecture et inclusion de fichiers du connecteur AJP (Ghostcat), déploiement Manager/Host-Manager vers RCE, et traversal par paramètre de chemin et contournement de contrainte de sécurité."
keywords:
  - mauvaise configuration de tomcat
  - ghostcat
  - AJP
  - tomcat manager
  - paramètres de chemin
---

# Tomcat et Java

Les serveurs d'applications Java (Apache Tomcat, et apparentés comme JBoss/WildFly, Jetty et GlassFish) présentent un ensemble de mauvaises configurations qui diffère des serveurs de fichiers Apache/nginx/IIS : un connecteur AJP secondaire, des applications de gestion embarquées qui déploient du code, et un traitement des chemins de servlet avec ses propres particularités. Identifiez Java à partir du cookie `JSESSIONID`, de `X-Powered-By: Servlet`, des pages d'erreur Tomcat par défaut et des applications `/manager`/`/examples`, puis parcourez cette liste de contrôle.

## Ce qu'il faut vérifier

- **Connecteur AJP (Ghostcat)** : le port AJP (classiquement `8009`) expose un protocole capable de lire ou d'inclure des fichiers de l'application web, atteignant la divulgation de code source et, avec un fichier téléversable, la RCE.
- **Applications Manager et Host-Manager** : des identifiants par défaut ou faibles sur `/manager/html` (et `/host-manager`) permettent de déployer un WAR, ce qui est une RCE directe.
- **Paramètres de chemin et normalisation** : le séparateur de paramètre de chemin `;` et la normalisation des servlets permettent à des chemins forgés de contourner les contraintes de sécurité et d'atteindre des servlets protégées.

## Pages

- **[AJP et Ghostcat](ajp-ghostcat.md)** : abuser du connecteur AJP pour la lecture de fichiers, l'inclusion et la RCE.
- **[Manager et Host-Manager](manager-and-host-manager.md)** : déployer un WAR pour l'exécution de code via les applications de gestion.
- **[Paramètres de chemin et normalisation](path-parameter-and-normalization.md)** : `..;/` et le traitement des chemins de servlet qui contourne les règles d'accès.

## Références

- Apache Tomcat documentation: AJP connector, Manager app, security
- OWASP WSTG: Testing for application platform configuration
