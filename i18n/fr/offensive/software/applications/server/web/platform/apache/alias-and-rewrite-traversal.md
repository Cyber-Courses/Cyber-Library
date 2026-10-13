---
order: 2
title: "Traversée via Apache Alias et mod_rewrite : mappage en dehors du répertoire prévu"
description: "Exploitation des règles Apache Alias, AliasMatch et mod_rewrite qui construisent des chemins du système de fichiers à partir de captures d'URL non contraintes, permettant à une requête de s'échapper du répertoire mappé vers le code source, la configuration ou des fichiers système."
keywords:
  - apache alias
  - AliasMatch
  - traversée mod_rewrite
  - RewriteRule
  - traversée de chemin
---

# Traversée via Alias et rewrite

Apache mappe des préfixes d'URL vers des répertoires avec `Alias`/`AliasMatch` et les réécrit avec `mod_rewrite`. Lorsqu'une règle construit un chemin du système de fichiers à partir d'un segment d'URL capturé sans contraindre `..`, une requête remonte vers le répertoire parent et atteint des fichiers hors du dossier prévu.

## Alias et AliasMatch

Un `Alias` simple n'est **pas** vulnérable au off-by-slash de nginx : Apache fait correspondre `Alias` sur des segments de chemin complets et normalise le chemin de l'URL (en réduisant les `../`) avant le traitement de l'alias, donc `/downloads../x` ne correspond pas à `Alias /downloads` et les segments de points décodés sont déjà résolus. Le vecteur Apache est plutôt un **`AliasMatch` ou un `RewriteRule` dont la capture regex est substituée dans un chemin du système de fichiers sans contrainte**, combiné à des séparateurs encodés qui survivent à la normalisation :

```apache
AliasMatch "^/files/(.*)$" "/srv/files/$1"
```

Ici `$1` n'est pas contraint. Comme Apache décode et normalise `../` dans le chemin avant la correspondance, un `/files/../../etc/passwd` littéral est d'abord réduit ; le payload fonctionnel repose sur des séparateurs **encodés** qui ne sont pas normalisés lorsque `AllowEncodedSlashes` vaut `On`/`NoDecode`, si bien que le `..%2f` encodé atteint la capture puis le système de fichiers :

```
GET /files/..%2f..%2f..%2fetc%2fpasswd HTTP/1.1
```

Sans `AllowEncodedSlashes NoDecode` (la valeur par défaut est `Off`, qui rejette `%2f`), vérifiez d'abord si le serveur accepte les slashes encodés ; s'il les rejette, ce chemin d'`AliasMatch` n'est pas exploitable et les cas de `RewriteRule` ci-dessous (ou un décalage de proxy) sont les voies restantes.

## mod_rewrite

Un `RewriteRule` qui compose un chemin à partir de segments capturés relève de la même classe, avec la même réserve : une règle qui s'applique au `REQUEST_URI` normalisé ne voit jamais de `../` littéral (il a été réduit), donc la traversée doit provenir d'une source non normalisée ou de séparateurs encodés :

```apache
RewriteRule ^/static/(.*)$ /srv/static/$1 [L]
```

Les variantes exploitables sont les règles qui s'appliquent à `%{THE_REQUEST}` (la ligne de requête brute, qui n'est ni décodée ni normalisée, donc les `../` bruts et encodés survivent tous deux jusqu'à la capture), les règles qui s'exécutent avec `AllowEncodedSlashes NoDecode` pour que `..%2f` atteigne `$1`, et les réécritures de proxy (`[P]`) qui construisent un chemin amont (le cas du proxy recoupe [reverse proxy et edge](../reverse-proxy-and-edge/index.md)).

## Exploitation

- Partez d'un préfixe mappé connu (via les URL d'assets ou un `.htaccess`/config divulgué), puis placez la traversée à l'intérieur du chemin capturé, en privilégiant les séparateurs encodés (`..%2f`, `%2e%2e%2f`, `%2e%2e/`) puisque Apache réduit les `../` littéraux avant la correspondance.
- Confirmez d'abord que le serveur accepte les slashes encodés (`AllowEncodedSlashes`) ; si `%2f` est rejeté, ce chemin est fermé et les cas `%{THE_REQUEST}`/proxy demeurent.
- Ciblez d'abord le parent (le code source et la configuration se trouvent généralement un niveau au-dessus : `../config.php`, `../.env`), puis remontez vers `/etc`.
- Combinez avec un encodage de type [double décodage](../iis/double-decode-and-unicode-traversal.md) quand un simple `..` encodé est filtré.
- Envoyez avec `curl --path-as-is` pour que le client ne pré-normalise pas le payload.

## Outils

- **curl --path-as-is** / Burp Intruder avec des wordlists de traversée.

## Références

- Apache httpd : mod_alias, mod_rewrite
- PortSwigger Web Security Academy : Path traversal
