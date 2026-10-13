---
order: 3
title: "Plateforme : attaquer le serveur web et la couche d'hébergement, par produit"
description: "Techniques offensives contre la plateforme web organisées comme le déroule un audit : expositions générales valables pour tout serveur, puis checklists de mauvaise configuration par produit pour Apache, nginx, IIS et Tomcat/Java, plus le comportement des reverse proxies et de l'edge."
keywords:
  - exploitation de serveur web
  - mauvaise configuration nginx
  - mauvaise configuration apache
  - mauvaise configuration iis
  - tomcat
  - reverse proxy
---

# Plateforme

La **plateforme** est le serveur web et la manière dont le service est hébergé et exposé : Apache, nginx, IIS ou un serveur d'application Java ; reverse proxies et CDN ; hôtes virtuels ; les règles qui mappent une URL vers un fichier ou un backend ; et les handlers qui décident si un fichier est servi, divulgué ou exécuté. Cette couche possède une classe de vulnérabilités distincte de la logique applicative ([Code](../code/index.md)) et du moteur de langage ([Runtime](../runtime/index.md)). Le test pour savoir si un bug appartient ici : *existerait-il encore si le code source de l'application et le runtime du langage étaient inoffensifs, parce que la faiblesse réside dans la configuration du serveur, la topologie de déploiement, ou la façon dont le serveur résout et route les requêtes ?*

## Comment ce domaine est organisé

La règle d'organisation de la bibliothèque consiste à diviser selon l'axe que vous identifiez *en premier* à une couche donnée, puis à affiner. Dans le Code applicatif, cet axe est la primitive dangereuse ; dans le Runtime, c'est la primitive puis le langage ou le moteur. À la couche plateforme, la chose que vous identifiez en premier est le **produit** : vous identifiez le serveur, puis déroulez la checklist de mauvaise configuration de ce produit. Ainsi, Plateforme est organisé par produit (Apache, nginx, IIS, Tomcat), ce qui est le même principe appliqué une couche plus loin, pas un écart par rapport à lui. Une unique section **générale** rassemble les expositions indépendantes du produit (un `.git` servi, listing de répertoire, sauvegardes, config divulguée) pour qu'elles ne soient pas répétées sous chaque produit, et une section **reverse proxy et edge** couvre le comportement qui relève du rôle de proxy plutôt que d'un produit en particulier.

Les primitives sous-jacentes traversent toujours chaque produit, donc chaque page est étiquetée avec sa primitive et le tableau ci-dessous offre une entrée orientée primitive, en plus de l'arbre orienté produit.

## Par primitive (référence croisée)

| Primitive | Où elle apparaît |
|-----------|------------------|
| Traversée de chemin / mappage | [alias/rewrite Apache](apache/alias-and-rewrite-traversal.md), [off-by-slash alias nginx](nginx/alias-off-by-slash.md), [normalisation de slash nginx](nginx/merge-slashes-and-normalization.md), [double décodage IIS](iis/double-decode-and-unicode-traversal.md), [décalage de normalisation du proxy](reverse-proxy-and-edge/normalization-mismatch.md) |
| Divulgation de source | [handler Apache](apache/handler-and-type-mapping.md), [MultiViews Apache](apache/multiviews-and-negotiation.md), [astuces NTFS IIS](iis/ntfs-filename-tricks.md), [VCS général](general/version-control-directories.md), [sauvegardes générales](general/backup-and-temporary-files.md) |
| Handler / upload vers exécution | [handler Apache](apache/handler-and-type-mapping.md), [FastCGI/PHP-FPM nginx](nginx/fastcgi-and-php-fpm.md), [handlers IIS](iis/handlers-and-web-config.md) |
| SSRF / routage | [proxy_pass variable nginx](nginx/variable-proxy-pass-ssrf.md), [proxy_pass et mauvais routage](reverse-proxy-and-edge/index.md), [exposition de l'origine](reverse-proxy-and-edge/origin-exposure.md) |
| RCE de déploiement | [Tomcat Manager](tomcat-and-java/manager-and-host-manager.md), [AJP/Ghostcat](tomcat-and-java/ajp-ghostcat.md) |
| Exposition d'information | [général](general/index.md) (endpoints de statut, VCS, sauvegardes, listing, config) |
| Confiance en-tête / identité | [confiance des en-têtes edge](reverse-proxy-and-edge/edge-header-trust.md) |

## Fingerprint d'abord

Avant les checklists, identifiez le serveur : en-têtes `Server`/`X-Powered-By`, pages d'erreur par défaut, noms de cookies (`JSESSIONID` = Java, `ASP.NET_SessionId` = IIS), ordre et casse des en-têtes, hashs de favicon et comportement face à des requêtes malformées. Le fingerprint sélectionne la section.

## Sections

- **[Général](general/index.md)** : expositions sur n'importe quel serveur, endpoints de statut, `.git`/VCS, sauvegardes, listing de répertoire, config divulguée.
- **[Apache](apache/index.md)** : mappage handler/type, traversée `Alias`/`mod_rewrite`, négociation MultiViews.
- **[nginx](nginx/index.md)** : off-by-slash d'`alias`, câblage FastCGI/PHP-FPM, SSRF via `proxy_pass` variable, normalisation de slash.
- **[IIS](iis/index.md)** : astuces de noms de fichiers NTFS, traversée double décodage/Unicode, handlers et `web.config`.
- **[Tomcat et Java](tomcat-and-java/index.md)** : AJP/Ghostcat, déploiement Manager, traversée par paramètre de chemin.
- **[Reverse proxy et edge](reverse-proxy-and-edge/index.md)** : décalage de normalisation edge/origine, exposition de l'origine, confiance des en-têtes edge.

## Références

- Documentation Apache httpd et nginx (références de configuration)
- PortSwigger Web Security Academy : Information disclosure, Access control, SSRF
