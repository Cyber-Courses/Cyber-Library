---
order: 3
title: "Mauvaise configuration de nginx : alias, FastCGI, proxy_pass et gestion des slashes"
description: "Mauvaises configurations de plateforme propres à nginx : la traversée off-by-slash location/alias, le câblage FastCGI/PHP-FPM qui exécute les uploads, la SSRF via proxy_pass variable, et les failles de merge_slashes/normalisation."
keywords:
  - mauvaise configuration nginx
  - alias off by slash
  - fastcgi_split_path_info
  - proxy_pass ssrf
  - merge_slashes
---

# nginx

nginx est à la fois un serveur statique, une passerelle FastCGI et un reverse proxy, et ses mauvaises configurations se regroupent autour de chaque rôle. Identifiez-le via `Server: nginx`, ses pages d'erreur par défaut et le comportement `location`/`proxy`, puis déroulez cette checklist. La config nginx est copiée-collée en permanence, donc la même poignée d'erreurs se répète d'une cible à l'autre.

## Ce qu'il faut vérifier

- **Off-by-slash d'`alias`** : un `location` sans slash final plus un `alias` avec en concatène les caractères suivants (y compris `..`) sur le chemin cible, divulguant le répertoire parent.
- **Câblage FastCGI/PHP-FPM** : `fastcgi_split_path_info` plus `cgi.fix_pathinfo` exécutant des fichiers qui contiennent simplement `.php` dans le chemin, et sockets FPM exposés.
- **`proxy_pass` variable** : un amont construit à partir de données de requête (`$host`, un argument) transforme le proxy en véhicule de SSRF.
- **Slash et normalisation** : `merge_slashes off`, gestion des slashes encodés, et correspondance de préfixe `location` en désaccord avec le système de fichiers ou un amont.

## Pages

- **[Off-by-slash d'alias](alias-off-by-slash.md)** : la traversée `location`/`alias` vers le répertoire parent.
- **[FastCGI et PHP-FPM](fastcgi-and-php-fpm.md)** : exécution d'upload via `fastcgi_split_path_info`, exposition de FPM et contrôle de `SCRIPT_FILENAME`.
- **[SSRF via proxy_pass variable](variable-proxy-pass-ssrf.md)** : sélection d'amont influencée par l'utilisateur et routage interne.
- **[merge_slashes et normalisation](merge-slashes-and-normalization.md)** : gestion des slashes et des préfixes qui contourne les règles basées sur `location`.

## Références

- Documentation nginx : location, alias, fastcgi_split_path_info, proxy_pass, merge_slashes
- Orange Tsai : recherche nginx off-by-slash et parser
