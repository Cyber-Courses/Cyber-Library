---
order: 1
title: "Mappage des handlers et des types Apache : exécuter ou divulguer les mauvais fichiers"
description: "Exploitation des erreurs Apache AddHandler/AddType/SetHandler : exécution via une extension interne, SetHandler sur un répertoire, divulgation de source par handler manquant, abus de surcharge .htaccess et exécution d'upload à double extension."
keywords:
  - AddHandler
  - AddType
  - SetHandler
  - htaccess
  - double extension
  - divulgation de source php
---

# Mappage des handlers et des types

Apache décide comment traiter un fichier à partir de son extension via `AddHandler`, `AddType` et `SetHandler`. Des mappages laxistes ou erronés provoquent deux défaillances opposées : des fichiers inertes se font **exécuter**, ou des fichiers exécutables se font **servir comme source**. Les deux sont de purs bugs de configuration.

## Exécution trop large (extension interne)

Apache fait correspondre `AddHandler`/`AddType` à *n'importe quelle* extension du nom de fichier, pas seulement la dernière :

```apache
AddHandler application/x-httpd-php .php
AddType application/x-httpd-php .php
```

Avec cela, `shell.php.jpg` est traité comme du PHP car `.php` apparaît dans le nom. Un filtre d'upload qui ne vérifie que l'extension finale (`.jpg`) laisse passer un fichier exécutable. C'est la moitié serveur de l'exécution d'upload à double extension.

Plus large encore, `SetHandler application/x-httpd-php` dans un `<Directory>`/`<Location>` couvrant un répertoire d'uploads ou de données exécute *chaque* fichier qui s'y trouve, quelle que soit l'extension.

## Exécution d'upload à double extension

Quand le handler correspond à une extension interne mais que le filtre d'upload ne valide que la dernière, une double extension satisfait les deux. Des extensions PHP alternatives contournent également les denylists :

```
shell.php.jpg      shell.phar.png     shell.pHp
shell.phtml        shell.pht          shell.php5     shell.php7
```

Associez à un polyglotte pour qu'un sniff de contenu ou `getimagesize` passe :

```
GIF89a;<?php system($_GET['c']); ?>
```

Localisez ensuite le fichier stocké (souvent via un [listing de répertoire](../general/directory-listing.md)) et demandez-le.

## Abus de surcharge .htaccess

Lorsque `AllowOverride` l'autorise et qu'un répertoire est accessible en écriture (un dossier d'uploads), un attaquant téléverse son propre `.htaccess` pour remapper une extension inoffensive vers le handler PHP :

```apache
AddType application/x-httpd-php .jpg
# now any uploaded .jpg in this directory runs as PHP
```

Téléverser ce `.htaccess` à côté d'un webshell en image est un classique de l'upload-vers-RCE quand l'upload direct d'un script est bloqué. Des variantes posent `php_value`/`php_flag` (par exemple `auto_prepend_file`) pour influencer le runtime depuis le répertoire.

## Divulgation de source par handler manquant

L'inverse expose le code source. Un emplacement où le handler PHP n'est pas câblé (un `RemoveHandler`/`SetHandler None`, un module désactivé après une mise à niveau, ou un mappage `.phps`/`text/plain`) renvoie les octets bruts du script :

```apache
<FilesMatch "\.php$">
    SetHandler None
</FilesMatch>
```

Demander n'importe quel `.php` divulgue alors le code source et les secrets, ce qui se voit couramment de façon transitoire juste après qu'une mise à niveau de PHP a retiré le handler.

## Exploitation

- Téléversez/placez `name.php.<ext-autorisée>` et demandez-le ; l'exécution confirme que le handler a correspondu au `.php` interne.
- Si les uploads atterrissent là où `AllowOverride` est activé, posez un `.htaccess` pour remapper une extension.
- Sondez les scripts connus à la recherche d'un `<?php` brut dans la réponse pour détecter une divulgation de source.

## Outils

- Burp (fuzzing d'upload) ; outillage de content-discovery ; constructeurs de polyglottes.

## Références

- Apache httpd : mod_mime (AddHandler/AddType), SetHandler, AllowOverride
- OWASP : Unrestricted file upload
