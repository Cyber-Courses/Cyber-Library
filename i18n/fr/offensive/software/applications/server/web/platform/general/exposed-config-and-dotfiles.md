---
order: 5
title: "Config et dotfiles exposés : .env, fichiers de configuration et métadonnées laissés lisibles par le web"
description: "Récupération d'identifiants et de secrets dans les fichiers de configuration et les dotfiles servis comme contenu statique : .env, config de framework, .htaccess, dotfiles de CI et d'éditeur, et fichiers de métadonnées cloud."
keywords:
  - exposition .env
  - divulgation de fichier de config
  - dotfiles
  - secrets
  - htaccess
---

# Config et dotfiles exposés

Les fichiers de configuration détiennent les secrets dont tout le reste dépend : identifiants de base de données, clés d'API, clés de signature du framework et tokens tiers. Quand ils se trouvent dans la racine web et que le serveur les renvoie comme texte statique (parce qu'il n'exécute que les `.php`, pas les `.env` ou `.yml`), une seule requête les exfiltre.

## Cibles à forte valeur

```
/.env                      # Laravel/Symfony/Node: DB creds, APP_KEY, API keys
/.env.local  /.env.prod    # environment variants
/config.php.txt /config.yml /settings.py /appsettings.json
/wp-config.php.bak         # backup variant is served as source
/.htaccess  /.htpasswd     # rules, and hashed creds
/docker-compose.yml  /Dockerfile   # service creds, build detail
/.aws/credentials  /.npmrc  /.pypirc   # cloud and registry tokens
/.DS_Store                 # macOS directory metadata -> filenames
/package.json  /composer.lock  /yarn.lock   # dependency inventory
```

`.env` est le fichier unique au rendement le plus élevé : il contient typiquement le DSN de la base de données, l'`APP_KEY`/`SECRET_KEY` du framework (qui forge cookies signés, sessions et tokens de réinitialisation de mot de passe), les identifiants de messagerie et les clés cloud.

## Les trouver et les lire

```
GET /.env HTTP/1.1
GET /.git/config HTTP/1.1
GET /.DS_Store HTTP/1.1
```

`.DS_Store` est binaire mais liste les noms de fichiers du répertoire ; parsez-le pour découvrir des fichiers non référencés, puis récupérez-les :

```bash
python3 ds_store_exp.py https://target/.DS_Store
```

Un `.htpasswd` servi fournit des hashs à casser hors ligne ; un `.htaccess` servi révèle les règles de réécriture, le périmètre d'authentification et les mappages de handlers qui éclairent les attaques [handler Apache](../apache/handler-and-type-mapping.md) et [alias/rewrite Apache](../apache/alias-and-rewrite-traversal.md).

## Exploitation

- Traitez un `APP_KEY`/`SECRET_KEY`/`machineKey` récupéré comme la fin de partie pour les artefacts signés : forgez sessions, cookies, JWT (quand HMAC avec ce secret) et ViewState.
- Les identifiants de base de données dans `.env` fonctionnent souvent contre un port de base de données exposé ou tunnelisable ; les tokens d'API/cloud permettent de pivoter hors de l'hôte.
- Les manifestes de dépendances (`composer.lock`, `package.json`) figent les versions exactes, que vous associez aux attaques connues de composants vulnérables dans les couches runtime et application.

## Outils

- **ffuf**/**feroxbuster** avec des wordlists de dotfiles ; **ds_store_exp** pour `.DS_Store` ; templates d'exposition **nuclei**.

## Références

- OWASP WSTG : Testing for information leakage ; review webserver metafiles
- SecLists : common dotfile and config wordlists
