---
order: 1
title: "Injection d'entité externe XML"
description: "Les parseurs XML qui résolvent les DTD et les entités externes permettent à un attaquant de lire des fichiers locaux, d'atteindre des services internes et d'amplifier l'entrée en déni de service."
keywords:
  - XXE
  - entité externe XML
  - injection de DTD
  - expansion d'entités
  - XXE hors bande
---

# XXE

L'injection d'entité externe XML abuse des parties de la spécification XML dont la plupart des applications n'ont jamais besoin : la **Document Type Definition** (DTD) et ses **entités**. Quand un parseur accepte du XML contrôlé par l'attaquant avec la résolution d'entités laissée activée, une déclaration telle que `<!ENTITY xxe SYSTEM "file:///etc/passwd">` fait récupérer au parseur une ressource externe et insérer son contenu dans le document partout où `&xxe;` apparaît. L'entrée reste du XML valide, donc elle passe par les points d'accès SOAP, les corps REST, les téléversements SVG et DOCX, SAML, et toute autre interface fondée sur XML.

Trois résultats découlent de cette seule primitive. La **divulgation** lit des fichiers locaux et les renvoie dans la réponse ou via un canal auxiliaire. La récupération **SSRF et hors bande** pointe l'entité vers des hôtes internes, des métadonnées cloud ou un serveur de l'attaquant, atteignant les ressources que le réseau du parseur peut voir. L'**amplification** imbrique des entités pour qu'un document minuscule se dilate en gigaoctets, épuisant la mémoire pour un déni de service.

Ce sous-arbre couvre la divulgation de fichier en bande, l'exfiltration hors bande, le SSRF, les attaques par expansion d'entités et les variantes aveugles qui fuitent des données via les erreurs du parseur quand rien n'est reflété.

## Sous-thèmes

- **[Aveugle](blind/index.md)** : quand le parseur résout les entités mais ne renvoie rien de parsé, les données sont récupérées via les messages d'erreur, le timing et les canaux hors bande.

## Pages

- **[Divulgation de fichier local](local-file-disclosure.md)** : une entité SYSTEM lit un fichier local et l'application reflète la valeur étendue dans sa réponse, renvoyant directement le contenu du fichier.
- **[Hors bande](out-of-band.md)** : quand rien n'est reflété, les entités paramètres et une DTD externe exfiltrent le contenu du fichier vers un serveur de l'attaquant via HTTP ou FTP.
- **[SSRF](ssrf.md)** : pointer une entité SYSTEM vers des hôtes internes ou des métadonnées cloud transforme un parseur XML en une primitive de server-side request forgery.
- **[Bombe XML](xml-bomb.md)** : des entités internes imbriquées s'étendent exponentiellement, transformant quelques kilooctets de XML en gigaoctets en mémoire et épuisant le parseur.

## Outils

- **XXEinjector** : automatisation de l'exploitation XXE y compris la récupération de fichiers OOB et basée sur les erreurs.
- **Burp Suite** : forge de charges utiles XXE et analyse de paramètres.
- **Burp Collaborator** : confirmation du XXE aveugle et hors bande.
- **oxml_xxe** : intégration de charges utiles XXE dans les téléversements OOXML et d'autres fichiers.

## Références

- PortSwigger Web Security Academy : XML external entity (XXE) injection
- OWASP : XML External Entity Prevention Cheat Sheet
