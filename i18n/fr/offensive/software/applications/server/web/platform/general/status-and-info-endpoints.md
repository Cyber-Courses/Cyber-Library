---
order: 1
title: "Endpoints de statut et d'info : exposition de server-status, server-info et stub_status"
description: "Récolte des tableaux de bord opérationnels laissés publics : mod_status et mod_info d'Apache, stub_status de nginx et pages de statut PHP-FPM, qui divulguent les requêtes en direct, les chemins internes et la configuration."
keywords:
  - server-status
  - mod_status
  - server-info
  - stub_status
  - statut php-fpm
---

# Endpoints de statut et d'info

Les serveurs web et les runtimes d'application embarquent des tableaux de bord opérationnels destinés à localhost ou à un réseau d'administration, mais fréquemment joignables depuis Internet. Ils divulguent le trafic en direct, les chemins internes, les listes de modules et la configuration, le tout sans authentification.

## mod_status et mod_info d'Apache

`mod_status` publie un tableau de bord serveur en direct ; quand `/server-status` n'est pas restreint, il montre les requêtes en cours de traitement, y compris les URL d'autres utilisateurs (avec leurs chaînes de requête), les IP clientes et les vhosts :

```
GET /server-status HTTP/1.1
GET /server-status?refresh=1 HTTP/1.1
```

`mod_info` sur `/server-info` déverse la configuration parsée complète : modules chargés, agencement des vhosts et directives de répertoire, soit une carte complète de la plateforme. Les deux fuient couramment parce qu'un bloc `Location` les restreignant était limité à un vhost alors que le module est global.

## stub_status de nginx

`ngx_http_stub_status_module` expose des compteurs de connexions et de requêtes :

```
GET /nginx_status HTTP/1.1
GET /status HTTP/1.1
GET /basic_status HTTP/1.1
```

Moins riche que celui d'Apache, mais il confirme nginx, révèle la charge, et la présence de l'endpoint signale une config copiée-collée qu'il vaut la peine de sonder davantage.

## Statut PHP-FPM et serveur d'application

PHP-FPM expose une page de statut de pool quand `pm.status_path` est défini et routé :

```
GET /status HTTP/1.1
GET /fpm-status HTTP/1.1
GET /fpm-ping HTTP/1.1
```

Elle divulgue le nom du pool FPM, le nombre de requêtes, les requêtes lentes et parfois les chemins des scripts en cours d'exécution. Des tableaux de bord équivalents existent pour d'autres stacks (stats uWSGI, statut Passenger) ; sondez les chemins courants et tous ceux référencés dans une config divulguée.

## Exploitation

- Demandez directement les chemins courants, et réessayez avec un `X-Forwarded-For: 127.0.0.1` ou un `Host: localhost` usurpé lorsque l'accès est filtré par IP ou par hôte au niveau du serveur.
- Sur `server-status`, interrogez avec `?refresh=1` et scrapez la table des requêtes au fil du temps pour capturer les tokens de session et les ID que d'autres utilisateurs passent dans les chaînes de requête.
- Réinjectez les chemins internes récupérés dans les techniques [FastCGI/PHP-FPM nginx](../nginx/fastcgi-and-php-fpm.md) et [handler Apache](../apache/handler-and-type-mapping.md).

## Outils

- **curl** / Burp ; **nuclei** dispose de templates pour ces expositions.

## Références

- Apache httpd : mod_status, mod_info
- nginx : ngx_http_stub_status_module ; PHP-FPM : pm.status_path
