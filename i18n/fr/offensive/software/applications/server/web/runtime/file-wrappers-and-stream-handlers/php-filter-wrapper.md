---
order: 1
title: "Wrapper php://filter : divulgation de source et transformation de contenu"
description: "Utiliser php://filter pour lire le code source PHP et des fichiers arbitraires via des filtres de conversion comme convert.base64-encode, en déjouant l'exécution par le moteur des fichiers inclus."
keywords:
  - php://filter
  - convert.base64-encode
  - divulgation de source
  - LFI
  - filtre de conversion
---

# Wrapper php filter

Le wrapper `php://filter` applique un ou plusieurs filtres de flux à une ressource au moment où elle est lue. Son usage offensif principal est la **divulgation de source** : lorsqu'une application inclut ou lit un chemin que vous influencez, envelopper la cible dans un filtre base64-encode renvoie les octets du fichier au lieu de les exécuter, de sorte que vous récupérez le source PHP qui ne ferait autrement que s'exécuter.

## Lire le source

Un `include`/`require` exécute normalement le PHP, donc un simple LFI d'un fichier `.php` produit sa sortie, pas son code. Encodez-le d'abord :

```
php://filter/convert.base64-encode/resource=config.php
php://filter/convert.base64-encode/resource=../../../../etc/passwd
```

Fourni au sink (par exemple `?page=php://filter/convert.base64-encode/resource=config`), l'include renvoie le base64 du fichier ; décodez-le hors ligne. Parce que le contenu est en base64, il survit à son passage dans le moteur PHP sans être interprété, ce qui est tout l'intérêt pour les cibles `.php`.

## Empilements de filtres utiles

Les filtres peuvent être chaînés et se répartissent en plusieurs familles :

- **convert.base64-encode / convert.base64-decode** : le cheval de bataille pour l'extraction sûre et pour le décodage.
- **convert.iconv.\*** : conversions de jeu de caractères (la base de la technique de RCE dans [chaînes de filtres vers RCE](filter-chains-to-rce.md)).
- **zlib.inflate / zlib.deflate** : (dé)compression, utile pour atteindre du contenu compressé ou comme étapes de chaîne.
- **string.rot13 / string.toupper** : transformations simples, parfois commodes pour esquiver des filtres naïfs sur l'entrée.

```
php://filter/read=convert.base64-encode/resource=index.php
php://filter/zlib.inflate/convert.base64-encode/resource=app.gz
```

## Notes d'exploitation

- Fonctionne contre des sinks comme `include`, `require`, `file_get_contents`, `readfile`, `fopen` et `highlight_file` où le chemin est influencé par l'attaquant ; cela ne requiert pas `allow_url_include`.
- Si l'application ajoute une extension (`$page . ".php"`), ciblez les fichiers en conséquence ou utilisez ensemble le path traversal et le wrapper.
- Le source que vous récupérez alimente toutes les autres attaques : il révèle des secrets, les sinks de désérialisation exacts et les classes disponibles pour des chaînes POP.
- Lorsqu'il vous faut l'exécution de code plutôt que la divulgation, escaladez avec les [chaînes de filtres vers RCE](filter-chains-to-rce.md), la [désérialisation phar](phar-deserialization.md) ou les [wrappers data et input](data-and-input-wrappers.md).

## Outils

- **curl** / Burp Repeater ; décodage base64 hors ligne.

## Références

- PHP manual: php://filter, stream filters
- PortSwigger Web Security Academy: LFI and PHP wrappers
