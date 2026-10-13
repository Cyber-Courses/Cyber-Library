---
title: "AJP et Ghostcat : lecture et inclusion de fichiers via le connecteur AJP de Tomcat"
description: "Exploiter un connecteur AJP de Tomcat exposé (Ghostcat) : lire ou inclure des fichiers de l'application web via des attributs AJP forgés, divulguer le code source et la configuration, et escalader vers une RCE avec un fichier téléversable."
keywords:
  - ghostcat
  - connecteur AJP
  - tomcat 8009
  - lecture de fichier
  - LFI vers RCE
---

# AJP et Ghostcat

Tomcat embarque un **connecteur AJP** (Apache JServ Protocol), classiquement sur le port `8009`, destiné à un Apache/nginx frontal qui transmet les requêtes sur un lien de confiance. AJP permet à l'appelant de définir des attributs de requête auxquels Tomcat accorde normalement sa confiance, y compris la servlet qui traitera la requête et le fichier auquel elle correspond. Lorsque le port AJP est joignable, un attaquant définit ces attributs directement. C'est la classe de bug Ghostcat.

**Un port 8009 joignable ne suffit pas à lui seul.** Le correctif Ghostcat (Tomcat 9.0.31, 8.5.51, 7.0.100 et versions ultérieures) a modifié les valeurs par défaut du connecteur AJP : le connecteur exige désormais un `secret` partagé (`secretRequired="true"` par défaut) et restreint les attributs de requête qu'un appelant peut définir via `allowedRequestAttributesPattern`. Sur un Tomcat corrigé et configuré par défaut, le connecteur rejette les attributs forgés, donc aucune lecture de fichier n'est possible. La technique s'applique lorsque la cible exécute une version antérieure au correctif, **ou** lorsqu'une version corrigée a vu son connecteur explicitement affaibli (`secretRequired="false"` sans `secret`, et un `allowedRequestAttributesPattern` permissif), ce que les opérateurs font parfois pour conserver un ancien montage de reverse proxy fonctionnel. Établissez la version/configuration avant de signaler une découverte.

## Lecture et inclusion de fichiers

En forgeant des attributs AJP (`javax.servlet.include.request_uri`, `...servlet_path`, `...path_info`), un attaquant fait en sorte que la servlet par défaut de Tomcat lise un fichier arbitraire situé sous la racine de l'application web et le renvoie, ou l'*inclue* comme s'il s'agissait d'une ressource de servlet :

- **Lecture** : divulguer tout fichier sous l'application web, y compris `WEB-INF/web.xml` (le descripteur de déploiement, la table des servlets, et souvent des identifiants) et des fichiers compilés/class et de configuration normalement protégés sous `WEB-INF`.
- **Inclusion** : traiter un fichier comme du JSP. Si l'attaquant parvient à déposer sur le serveur un fichier contenant du JSP (un téléversement stocké sous la racine web, même en `.txt`/image), son inclusion exécute le JSP, ce qui donne une **RCE**.

```bash
# Metasploit: auxiliary/admin/http/tomcat_ghostcat  (file read)
# or the standalone PoC:
python3 ghostcat.py -p 8009 -f WEB-INF/web.xml target
```

## Comment le repérer

- Scanner le port `8009/tcp` (AJP). Un connecteur AJP ouvert qui n'est pas filtré par pare-feu vers le proxy frontal est la condition préalable, mais pas une preuve (voir le filtre version/configuration ci-dessus).
- Identifier la version de Tomcat (pages d'erreur, en-tête `Server`, `/docs`) pour juger si elle est antérieure au correctif Ghostcat.
- Confirmer l'exploitabilité en lisant `WEB-INF/web.xml` ; une lecture réussie prouve la primitive de lecture de fichier et cartographie les servlets de l'application ainsi que tout secret embarqué. Un connecteur exigeant un secret se contente de ne rien renvoyer d'utile.

## Exploitation

- Lire `WEB-INF/web.xml`, puis les fichiers de configuration et de classes référencés, pour récupérer des identifiants (BDD, application de gestion), les mappings de servlets et des secrets.
- Pour une RCE, trouver une fonctionnalité qui stocke du contenu de l'attaquant sous la racine web (téléversement, avatar, journal, cache), puis utiliser l'*inclusion* AJP pour l'exécuter comme du JSP.
- Les identifiants récupérés dans `web.xml`/la configuration débloquent fréquemment la [Manager app](manager-and-host-manager.md) pour une RCE plus propre via déploiement de WAR.

## Outils

- PoC **ghostcat**, Metasploit `tomcat_ghostcat`, scripts AJP de **nmap**.

## Références

- Apache Tomcat: AJP connector documentation and security guidance
- Ghostcat technical writeups
