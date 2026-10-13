---
order: 3
title: "Handlers IIS et web.config : mappages d'exécution, divulgation de source et abus de configuration"
description: "Exploitation des mappages de handlers IIS et de web.config : handlers d'extension qui exécutent les uploads, divulgation de source par handler manquant et abus de web.config d'upload par répertoire."
keywords:
  - handlers iis
  - web.config
  - mappage de handler
  - upload rce
  - divulgation de source
---

# Handlers et web.config

IIS mappe les extensions de requête vers des handlers (le pipeline ASP.NET, le handler de fichiers statiques, CGI/FastCGI) et lit un `web.config` par répertoire qui peut modifier ces mappages. Les mauvaises configurations ici exécutent des uploads, divulguent du code source ou laissent un attaquant reconfigurer un répertoire dans lequel il peut écrire.

## Mappage de handler et exécution d'upload

Si un répertoire qui accepte les uploads est servi par un handler qui exécute l'extension téléversée, l'upload est une RCE. Les extensions propres à IIS pour contourner les denylists incluent :

```
shell.aspx   shell.asp   shell.ashx   shell.asmx   shell.cer   shell.soap
```

`.cer` et `.asa`/`.asax` ont historiquement été mappés à des handlers exécutants sur des serveurs mal configurés. Combinez avec les [astuces de noms de fichiers NTFS](ntfs-filename-tricks.md) (`shell.aspx;.jpg`, point final) pour glisser une extension exécutable devant un filtre d'upload pendant qu'IIS l'exécute quand même. L'astuce du point-virgule `;` (`file.asp;.jpg`) faisait exécuter aux IIS plus anciens la partie avant le point-virgule.

## web.config comme vecteur d'exécution

`web.config` est lu par répertoire. Lorsque les uploads atterrissent dans un répertoire sans verrou parent et que l'application ne le bloque pas, téléverser un `web.config` fabriqué peut :

- enregistrer un handler qui exécute une extension inoffensive dans ce répertoire, ou
- définir un bloc ASP classique ou une entrée `httpHandlers`/`handlers` qui exécute du code,
- et dans certaines configurations, le `web.config` lui-même exécute des server-side includes ou une syntaxe d'expression lors de son traitement.

Cela transforme « téléverser n'importe quel fichier » en RCE même quand les extensions de script sont bloquées, de façon analogue à l'upload de `.htaccess` d'Apache.

## Divulgation de source

- Demandez les scripts exécutables via `::$DATA` / point final (voir [astuces de noms de fichiers NTFS](ntfs-filename-tricks.md)) pour obtenir la source brute plutôt que l'exécution.
- Un répertoire où le handler ASP.NET n'est pas câblé (handler de fichiers statiques servant `.aspx`) renvoie la source ; cela apparaît après des déploiements mal configurés ou quand un handler est retiré.
- Le `web.config` lui-même, s'il est servi comme contenu statique (handler mal configuré), divulgue les chaînes de connexion, les machine keys et l'agencement des handlers. Une `machineKey` divulguée permet la [forge de ViewState](../../runtime/insecure-deserialization/dotnet-deserialization.md).

## Exploitation

- Fuzzez les uploads avec les extensions exécutables IIS plus des astuces de noms de fichiers ; confirmez l'exécution avec un marqueur inoffensif.
- Lorsque les uploads le permettent, essayez de poser un `web.config` pour remapper ou exécuter.
- Sondez les `.aspx`/`.asmx` connus pour la source via `::$DATA`, et demandez `/web.config` (et `::$DATA`) pour la divulgation de config et la machine key.

## Outils

- Burp (fuzzing d'upload) ; **IIS-ShortName-Scanner** pour trouver les scripts cachés ; outillage de content-discovery.

## Références

- Microsoft IIS : handler mappings, web.config, request filtering
- OWASP : Unrestricted file upload
